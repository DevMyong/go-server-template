---
이 문서는 자주 업데이트됩니다. 마지막 수정: 2025-10-02

---
## 📄 2. architecture.md - 아키텍처 원칙

# Go 서버 템플릿 아키텍처 가이드
## Clean Architecture 개요

```markdown
외부 계층 (Frameworks & Drivers)
↓
인터페이스 계층 (Interface Adapters)
↓
응용 계층 (Application Business Rules)
↓
도메인 계층 (Enterprise Business Rules)
```

**핵심 원칙**: 의존성은 항상 안쪽(도메인)을 향함

---

## 레이어별 책임

### Handler Layer (외부)
**위치**: `internal/handler/`

**책임**:
- HTTP 요청/응답 처리
- 입력 검증 (기본)
- 에러를 HTTP 상태코드로 변환
- Thin layer (비즈니스 로직 없음)

**금지**:
- ❌ 비즈니스 로직
- ❌ 직접 DB 접근
- ❌ 복잡한 데이터 변환

**예시**:
```go
type UserHandler struct {
    userService UserService // 인터페이스
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    // 1. 요청 파싱
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, err)
        return
    }
    
    // 2. 기본 검증
    if err := validate.Struct(req); err != nil {
        respondError(w, http.StatusBadRequest, err)
        return
    }
    
    // 3. Service 호출 (비즈니스 로직 위임)
    user, err := h.userService.CreateUser(r.Context(), req.ToModel())
    if err != nil {
        respondError(w, toHTTPStatus(err), err)
        return
    }
    
    // 4. 응답
    respondJSON(w, http.StatusCreated, user)
}
```

---

Service Layer (응용)
위치: internal/service/
책임:

비즈니스 로직 구현
트랜잭션 관리
여러 Repository 조합
도메인 규칙 검증

의존:

Repository 인터페이스 (구현체 아님!)
Model

예시:
```go
type UserService struct {
    userRepo    UserRepository    // 인터페이스
    emailSender EmailSender       // 인터페이스
    cache       CacheRepository   // 인터페이스
}

func (s *UserService) CreateUser(ctx context.Context, user *model.User) error {
    // 1. 비즈니스 규칙 검증
    if exists, _ := s.userRepo.ExistsByEmail(ctx, user.Email); exists {
        return ErrUserAlreadyExists
    }
    
    // 2. 비즈니스 로직
    user.Password = hashPassword(user.Password)
    user.Status = "pending"
    
    // 3. Repository 호출
    if err := s.userRepo.Create(ctx, user); err != nil {
        return fmt.Errorf("failed to create user: %w", err)
    }
    
    // 4. 부가 작업
    go s.emailSender.SendWelcomeEmail(user.Email)
    
    // 5. 캐시 무효화
    s.cache.Delete(ctx, "users:*")
    
    return nil
}
```

---

Repository Layer (인터페이스)
위치: internal/repository/
책임:

데이터 접근 추상화
CRUD 연산
쿼리 구현

의존:

Model만
DB 드라이버 (pkg/database)

인터페이스 정의 위치: Service에서 정의!

```go
// ✅ Good: service/user_service.go
type UserRepository interface {
    Create(ctx context.Context, user *model.User) error
    FindByID(ctx context.Context, id int64) (*model.User, error)
    FindByEmail(ctx context.Context, email string) (*model.User, error)
    ExistsByEmail(ctx context.Context, email string) (bool, error)
}

// ✅ Good: repository/user_repository.go
type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *model.User) error {
    return r.db.WithContext(ctx).Create(user).Error
}
```

Model Layer (도메인)
위치: internal/model/
책임:

도메인 엔티티 정의
도메인 규칙 (간단한 것)
Validation tag

의존: 없음 (가장 안쪽 계층)
```go
type User struct {
    ID        int64     `json:"id" db:"id"`
    Email     string    `json:"email" db:"email" validate:"required,email"`
    Password  string    `json:"-" db:"password" validate:"required,min=8"`
    Name      string    `json:"name" db:"name" validate:"required"`
    Status    string    `json:"status" db:"status"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

// 도메인 메서드 (간단한 것만)
func (u *User) IsActive() bool {
    return u.Status == "active"
}
```

---

의존성 주입 패턴
Wire-up (main.go)
```go
func main() {
    // 1. 인프라 초기화
    db := database.NewMySQL(config.MySQL)
    cache := database.NewRedis(config.Redis)
    logger := logger.New(config.Logger)
    
    // 2. Repository 생성
    userRepo := repository.NewUserRepository(db)
    
    // 3. Service 생성 (인터페이스 주입)
    userService := service.NewUserService(userRepo, cache, logger)
    
    // 4. Handler 생성
    userHandler := handler.NewUserHandler(userService)
    
    // 5. 라우터 설정
    router := setupRouter(userHandler)
    
    // 6. 서버 시작
    server := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }
    server.ListenAndServe()
}
```

---

패키지 구조 상세
```markdown
internal/
├── config/           # 설정 로딩 및 검증
│   ├── config.go
│   └── validator.go
├── handler/          # HTTP 핸들러
│   ├── user_handler.go
│   ├── response.go   # 공통 응답 헬퍼
│   └── request.go    # 공통 요청 DTO
├── middleware/       # HTTP 미들웨어
│   ├── logger.go
│   ├── recovery.go
│   ├── cors.go
│   └── request_id.go
├── service/          # 비즈니스 로직
│   ├── user_service.go
│   └── user_service_test.go
├── repository/       # 데이터 접근
│   ├── user_repository.go
│   └── user_repository_test.go
└── model/            # 도메인 모델
    └── user.go

pkg/
├── database/         # DB 연결 관리
│   ├── mysql.go
│   ├── mongodb.go
│   └── redis.go
├── logger/           # 로깅 유틸
│   └── logger.go
└── httpclient/       # HTTP 클라이언트
    └── client.go
```

---

에러 처리 전략
도메인 에러 (model/)
```go
var (
    ErrUserNotFound      = errors.New("user not found")
    ErrUserAlreadyExists = errors.New("user already exists")
    ErrInvalidPassword   = errors.New("invalid password")
)
```

에러 래핑 (repository, service)
```go
func (r *repository) FindByID(ctx context.Context, id int64) (*model.User, error) {
    var user model.User
    err := r.db.WithContext(ctx).First(&user, id).Error
    if err == gorm.ErrRecordNotFound {
        return nil, model.ErrUserNotFound
    }
    if err != nil {
        return nil, fmt.Errorf("failed to find user: %w", err)
    }
    return &user, nil
}
```

HTTP 변환 (handler)
```go
func toHTTPStatus(err error) int {
    switch {
    case errors.Is(err, model.ErrUserNotFound):
        return http.StatusNotFound
    case errors.Is(err, model.ErrUserAlreadyExists):
        return http.StatusConflict
    case errors.Is(err, model.ErrInvalidPassword):
        return http.StatusUnauthorized
    default:
        return http.StatusInternalServerError
    }
}
```

---

트랜잭션 관리
Service에서 처리
```go
func (s *UserService) CreateUserWithProfile(ctx context.Context, user *model.User, profile *model.Profile) error {
    // Repository에 트랜잭션 메서드 추가
    return s.userRepo.WithTransaction(ctx, func(tx *gorm.DB) error {
        if err := s.userRepo.CreateWithTx(ctx, tx, user); err != nil {
            return err
        }
        profile.UserID = user.ID
        if err := s.profileRepo.CreateWithTx(ctx, tx, profile); err != nil {
            return err
        }
        return nil
    })
}
```

---

테스트 전략
Repository 테스트 (통합)
```go
// 실제 DB 사용 (docker-compose)
func TestUserRepository_Create(t *testing.T) {
    db := setupTestDB(t)
    repo := NewUserRepository(db)
    
    user := &model.User{Email: "test@example.com"}
    err := repo.Create(context.Background(), user)
    
    assert.NoError(t, err)
    assert.NotZero(t, user.ID)
}
```

Service 테스트 (단위)
```go
// Mock Repository 사용
func TestUserService_CreateUser(t *testing.T) {
    mockRepo := new(MockUserRepository)
    service := NewUserService(mockRepo, nil, nil)
    
    mockRepo.On("ExistsByEmail", mock.Anything, "test@example.com").
        Return(false, nil)
    mockRepo.On("Create", mock.Anything, mock.Anything).
        Return(nil)
    
    err := service.CreateUser(context.Background(), &model.User{
        Email: "test@example.com",
    })
    
    assert.NoError(t, err)
    mockRepo.AssertExpectations(t)
}
```

Handler 테스트 (통합)
```go
// httptest 사용
func TestUserHandler_CreateUser(t *testing.T) {
mockService := new(MockUserService)
handler := NewUserHandler(mockService)

    mockService.On("CreateUser", mock.Anything, mock.Anything).
        Return(&model.User{ID: 1}, nil)
    
    req := httptest.NewRequest("POST", "/users", strings.NewReader(`{"email":"test@example.com"}`))
    rec := httptest.NewRecorder()
    
    handler.CreateUser(rec, req)
    
    assert.Equal(t, http.StatusCreated, rec.Code)
}
```

---

참고 자료:
- Clean Architecture (Robert C. Martin)
- Go Clean Arch Example