# Basics
## 2023-01-14 Sat
### 애플리케이션 설정
* 코드 중 환경에 따라 다르게 동작하거나 노출되지 않아야하는 코드가 있으며, 이러한 류의 코드는 설정 파일을 통해 쉽게 관리할 수 있다.
* **설정 파일은 런타임 도중에 지속적으로 변경되는 것이 아닌, 애플리케이션이 실행되는 과정에서 로드되어 값들을 정의하는 방식으로 사용**된다.
  * 또한, 설정 파일은 일반적으로 XML이나 JSON, YAML 또는 환경 변수 등의 형태로 사용된다.
* 이러한 설정 파일은 그 유형에 따라 다시 다음과 같은 두 가지로 분류될 수 있다.
  1. 코드 기반 설정: **XML이나 JSON 또는 YAML과 같은 설정 파일이며, 주로 포트 번호와 같이 노출되어도 상관 없는 정보를 설정하는 용도로 사용**된다.
  2. 환경 변수: **비밀번호 또는 API Key 등 노출되지 않아야 하는 민감한 정보를 위해 사용**된다.
* 이 때, **NodeJS를 활용하는 애플리케이션의 경우 일반적으로 `config` 모듈을 활용하여 설정 파일을 관리**한다. 
* 또한, `config` 모듈과 함께 설정 정보를 YAML 파일로 관리하는 경우를 예로 들어 다음과 같은 파일들을 정의할 수 있다.
  1. default.yml: 기본 설정이며, 환경에 무관하게 적용되는 설정을 명시한다.
  2. development.yml: 개발 환경에 필요한 정보이며, 기본 설정인 default.yml의 내용에 더해 적용된다.
  3. production.yml: 운영 환경에 필요한 정보이며, 기본 설정인 default.yml의 내용에 더해 적용된다.
* 상술한 **설정 파일들은 프로젝트 최상위에 `config/*.yml` 형태로 작성하여 어떠한 TS 파일에서든지 `config` 모듈을 통해 다음과 같이 접근**할 수 있다.
```yaml
# default.yml
server:
  port: 3000
```
```typescript
import * as config from 'config';

async function bootstrap() {
  const logger = new Logger();
  const app = await NestFactory.create(AppModule);

  // config 파일에서 server 정보만을 가져온다.
  const server = config.get('server');
  await app.listen(server.port);

  logger.log(`Application running on port ${server.port}`);
}
bootstrap();
```
* 이를 활용하여 데이터베이스 관련 설정 역시 YAML 파일로 다음과 같이 관리할 수 있다.
```yaml
# default.yml
db:
  type: 'postgres'
  port: 35432
  database: board-app
  synchronized: false

# development.yml
db:
  host: '127.0.0.1'
  username: 'postgres'
  password: 'my-first-nest-pw'
  # 아래와 같이 default.yml의 내용과 겹치는 경우, development.yml의 우선 순위가 더 높게 적용되어 true로 설정된다.
  synchronized: true
```
* 이러한 설정 정보는 TypeOrmConfig 파일을 다음과 같이 수정하여 적용할 수 있다.
  * 이 때, 민감한 정보를 환경 변수로 관리하고자 하는 경우 `process.env.[환경변수명]`을 우선 적용하도록 다음과 같이 작성할 수 있다.
```typescript
const logger = new Logger('TypeOrmConfig');
const databaseConfig = config.get('db');
logger.debug(`Database Config: ${JSON.stringify(databaseConfig)}`);

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: databaseConfig.type,
  host: process.env.RDS_HOSTNAME || databaseConfig.host,
  port: process.env.RDS_PORT || databaseConfig.port,
  username: process.env.RDS_USERNAME || databaseConfig.username,
  password: process.env.RDS_PASSWORD || databaseConfig.password,
  database: process.env.RDS_DB_NAME || databaseConfig.database,
  entities: [__dirname + '/../**/*.entity.{js,ts}'],
  // synchronize 옵션은 데이터 유실을 방지하기 위해 운영 환경에서는 반드시 false로 지정해두어야 한다.
  synchronize: databaseConfig.synchronize,
};
```
* 반면, 파일이 특정한 단 하나의 config property만을 사용하는 경우에는 다음과 같이 `config.get('config.property');` 형태로 입력할 수도 있다.
```typescript
const logger = new Logger('JwtStrategy');
// 아래와 같이 jwt 전체가 아닌 jwt.secret 정보만을 가져올 수도 있다. 
const jwtSecret = config.get('jwt.secret');
logger.debug(`Jwt Strategy: ${JSON.stringify(jwtSecret)}`);

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private authRepository: AuthRepository) {
    super({
      secretOrKey: process.env.JWT_SECRET || jwtSecret,
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
    });
  }
  // 생략
}
```