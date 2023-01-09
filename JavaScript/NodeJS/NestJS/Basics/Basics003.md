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
* 