# Basics
## 2023-01-09 Mon
### NestJS를 활용하여 인증 서비스 구현하기
* 인증은 일반적으로 별도의 도메인으로 취급되므로, nest cli 명령어를 활용하여 별도의 모듈을 생성한다.

### 간단한 비밀번호 유효성 검증 로직 추가하기
* 너무 짧거나 긴 비밀번호를 방지하고 싶은 경우 등, 기초적인 유효성 검증은 데코레이터를 활용하여 dto에 다음과 같이 작성할 수 있다.
  * 물론 컨트롤러의 핸들러 메소드 매개 변수에 명시한 데코레이터는 `@Body(ValidationPipe)` 와 같은 형태로 수정해두어야 한다.
  * 이렇듯 ValidationPipe를 명시하는 것으로 핸들러 메소드가 호출되기 전에 dto의 유효성 검증을 수행할 수 있다. 
```typescript
export class AuthCredentialDto {
    @IsString()
    @MinLength(4)
    @MaxLength(20)
    username: string;

    @IsString()
    @MinLength(4)
    @MaxLength(20)
    @Matches(/^[a-zA-Z0-9]*$/, {
        message: 'password only accepts English and number!',
    }) // 영문 또는 숫자만을 허용, 유효성 검증 실패시 message에 명시된 값이 출력된다.
    password: string;
}
```

### 중복된 사용자 이름 체크하기
* TypeORM을 사용하여 중복된 사용자 이름을 검증하는 방법은 크게 다음과 같이 분류할 수 있다.
  1. `findOne()` 메소드를 활용하여 같은 사용자명이 존재하는지 확인한 후에 데이터를 저장
  2. 또는 데이터베이스 차원에서 중복된 사용자명이 존재하는 경우에 에러를 던지도록 구현
* 이 중 첫 번째 방식은 불필요한 DB IO가 발생하므로, 일반적으로는 다음과 같이 두 번째 방법을 사용하는 것이 바람직하다.
  * 이렇듯 TypeORM에서는 `@Unique()` 데코레이터를 활용하여 이러한 기능을 간단하게 구현할 수 있다.
```typescript
@Entity()
@Unique(['username']) // @Unique() 데코레이터의 인자에 중복을 체크할 컬럼을 배열 형태로 전달한다.
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  password: string;
}
```
* 상술한 어노테이션은 내부적으로는 테이블에 UQ 제약 조건을 거는 식으로 동작하며, 다음과 같은 특이 사항이 존재한다.
  1. 테이블에 존재하지 않는 컬럼명을 명시할 경우, 정상적으로 동작하지 않는다.
  2. 이미 작성한 `@Unique()` 데코레이터를 제거하는 경우, 서버가 재시작될 때 해당 제약 조건을 제거한다.
* 이 때, 중복된 사용자명을 전달할 경우 다음과 같이 500 에러가 발생하므로 반드시 이를 핸들링해주어야 한다.
```shell
[Nest] 2796  - 01/09/2023, 7:54:23 PM   ERROR [ExceptionsHandler] duplicate key value violates unique constraint "UQ_78a916df40e02a9deb1c4b75edb"
QueryFailedError: duplicate key value violates unique constraint "UQ_78a916df40e02a9deb1c4b75edb"
```
* 이러한 에러는 pg에서 발생한 예외를 적절히 핸들링하지 않아 컨트롤러까지 에러가 전파되는 것으로, 코드를 통해 다음과 같이 간단하게 수정할 수 있다.
```typescript
async createUser(dto: AuthCredentialDto): Promise<void> {
  const { username, password } = dto;
  const user = this.create({ username, password });

  try {
    await this.save(user);
  } catch (e) {
    if (e.code === '23505')
      throw new ConflictException(`username(${username}) is already exists.`);

    throw new InternalServerErrorException();
  }
}
```

## 2023-01-10 Tue
### 비밀번호 암호화하기
* 비밀번호를 데이터베이스에 저장하는 방법은 크게 다음과 같이 분류할 수 있다.
  1. 평문으로 저장하기: 최악의 방법이며, 개발자라면 절대 하지 말아야 할 방식에 해당한다.
  2. 암호화 키를 활용하여 암호화하기: 양방향 암호화에 해당하며, 암복호화를 모두 하나의 암호화 키와 암호화 알고리즘 조합을 사용한다.
  3. SHA256 등을 통해 단방향 암호화하기: 단방향 암호화에 해당하며, 알고리즘을 활용하여 해시된 값을 저장한다.
  4. 솔트와 단방향 암호화를 조합하기: 사용자가 입력한 비밀번호에 임의의 문자열인 솔트를 더한 후에 알고리즘을 적용하여 해시된 값을 저장한다.
* 이 때, 양방향 암호화의 경우 대부분의 알고리즘이 노출되어 있으므로 암호화 키가 노출되었을 경우 보안에 취약해질 수 있다는 특징이 있다.
  * 반면, 단방향 암호화의 경우 복호화할 수 없으나 많은 사용자가 적용하는 비밀번호의 형태가 유사하므로 레인보우 테이블 방식을 통해 공격당할 가능성이 존재한다.
* 상술한 방식 중 가장 적절한 것은 솔트와 단방향 암호화를 조합한 것으로, 이는 bcryptjs와 같은 라이브러리를 활용하여 다음과 같이 간단하게 구현이 가능하다.
  * 이 경우, 비밀번호에 사용되는 salt는 매 번 달라지므로 같은 비밀번호를 사용하는 사용자들도 서로 다른 값을 데이터베이스에 저장하게 된다.
```typescript
async createUser(dto: AuthCredentialDto): Promise<void> {
  const { username, password } = dto;

  const salt = await bcrypt.genSalt(); // bcryptjs를 활용하여 salt를 생성한다.
  const hashedPassword = await bcrypt.hash(password, salt);  // 생성된 salt를 활용하여 해시한다.
  const user = this.create({ username, password: hashedPassword }); // 해시된 값을 데이터베이스에 저장한다.

  try {
    await this.save(user);
  } catch (e) {
    if (e.code === '23505')
      throw new ConflictException(`username(${username}) is already exists.`);

    throw new InternalServerErrorException();
  }
}
```

### 로그인 기능 구현하기
* 상술한 내용을 토대로, 사용자에게 username과 비밀번호를 전달받았을 때 이를 bcryptjs를 활용하여 검증하는 코드는 다음과 같이 간단하게 구현이 가능하다.
  * 이 때, `bcrypt.compare()` 메소드에 전달할 인자의 순서를 바꾸면 로그인에 실패하므로 순서에 주의하여 반드시 평문을 먼저 전달해야 한다.
```typescript
async signIn(dto: AuthCredentialDto): Promise<string> {
  const { username, password } = dto;
  const user = await this.authRepository.getUserByUsername(username);
  if (user && (await bcrypt.compare(password, user.password)))
    return 'login success';

  throw new UnauthorizedException('login failed');
}
```

## 2023-01-11 Wed
### JWT란?
* **JWT는 로그인된 사용자에게 발행하기 위한 토큰이며, 정보를 디지털 서명된 JSON 객체 형태로 안전하게 전송하기 위한 표준**이기도 하다. 
  * 이러한 JWT는 사용자의 정보를 안전하게 전달하면서도 권한을 체크하는 데에 유용하게 사용될 수 있다.
* JWT는 `헤더.페이로드.시그니쳐`와 같이 크게 셋으로 분류되는 구조를 갖고, 각 구조는 다음과 같은 특징을 갖는다.
  1. 헤더: 타입이나 해싱 알고리즘 등 토큰 자체에 대한 메타데이터를 갖는다. 
  2. 페이로드: 사용자 정보나 만료 기간, 또는 주제와 같은 정보를 갖는다.
  3. 시그니쳐: 토큰의 유효성을 검증하기 위한 서명으로, 헤더와 페이로드 및 서명 알고리즘과 암호화 키를 활용하여 생성된다.

### JWT 활용 흐름
* JWT를 사용하는 경우, 인증 절차는 크게 다음과 같이 분류할 수 있다.
  1. 사용자가 로그인한다.
  2. 로그인 정보가 맞는 경우, 사용자 정보를 암호화한 JWT 토큰을 발급하여 사용자에게 반환한다.
  3. 서버는 토큰을 보관하며, 사용자가 요청시 제출한 토큰에 대한 유효성 검증에 이를 활용한다.
* 또한 사용자는 발급받은 토큰을 보관하며, 인증이 필요한 경우에 다음과 같은 절차를 거친다.
  1. 요청시 자신이 갖고 있던 토큰 정보를 헤더에 포함시킨다.
  2. 서버는 JWT를 활용하여 토큰을 분석한 후, 헤더 및 페이로드에 자신이 갖는 암호화 정보를 활용하여 시그니쳐를 다시 생성하고 이를 사용자의 값과 비교한다.
  3. 비교 결과가 참인 경우, 사용자의 요청을 처리한 후 결과를 반환한다.

### NestJS에서 JWT 사용하기
* NestJS 프로젝트에서 JWT와 더불어 passport를 사용하기 위해서는 우선 다음과 같이 필요한 모듈을 설치한다.
  * 명령어: `npm i @nestjs/jwt @nestjs/passport passport passport-jwt`
* 이후 AuthModule을 다음과 같이 수정하여 애플리케이션 전역에서 JWT와 passport 모듈을 사용할 수 있도록 수정하되 다음과 같은 옵션을 입력한다.
  1. secret: 토큰의 시그니쳐 구간을 생성하기 위해 사용되는 평문을 의미한다. 
  2. signOptions.expiresIn: 토큰의 만료 기간을 정의한다.
```typescript
@Module({
  imports: [
    PassportModule.register({
      defaultStrategy: 'jwt',
    }),
    JwtModule.register({
      secret: 'my-super-secret',
      signOptions: {
        expiresIn: 60 * 60,
      },
    }),
    TypeOrmExModule.forCustomRepository([AuthRepository]),
  ],
  providers: [AuthService],
  controllers: [AuthController],
})
export class AuthModule {}
```
* 이렇듯 필요한 모든 모듈을 애플리케이션에 import한 경우, AuthService를 다음과 같이 수정하여 인증 성공시 토큰을 반환할 수 있도록 한다.
```typescript
// 더 이상 login succeed라는 문자열을 반환하지 않으므로, 함수의 반환형을 수정한다.
async signIn(dto: AuthCredentialDto): Promise<{ accessToken: string }> {
  const { username, password } = dto;
  const user = await this.authRepository.getUserByUsername(username);
  if (user && (await bcrypt.compare(password, user.password)))
    return this.#createToken(username);

  throw new UnauthorizedException('login failed');
}

// jwt 토큰의 payload에는 원하는 내용을 추가하되, 민감한 정보는 담지 않는 것이 바람직하다.
#createToken(username) {
  const payload = { username };
  const result = this.jwtService.sign(payload);
  return { accessToken: result };
}
```