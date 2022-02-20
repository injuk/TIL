# CodingTest
## 2022-02-20 Sun

## 문자열(String)
### 문자열 읽어들이기
* 주로 백준에서 사용하는 방식이다.
* 공백이 포함된 문자는 반드시 nextLine을 사용하자.
```
Scanner scanner = new Scanner(System.in);
String input = scanner.next();

// String input = scanner.nextLine();
```

### 문자열의 모든 문자 순회하기
* 향상된 for 문이 기본 for 문보다 가독성이 더 좋다.
* 향상된 for 문의 우항에는 Iterable 또는 배열을 배치한다.
```
String test = "Hello World!";
for(int i = 0; i < test.length(); i++)
System.out.println(test.charAt(i));
// 또는
String temp = "Hello World!";
for(char c : temp.toCharArray())
System.out.println(c);
```

### 문자열을 공백 기준으로 나누기
* split을 사용할 수 있지만, 성능이 subString 등의 메소드보다 훨씬 안좋다고 한다.
  * split은 javasctip의 split과 달리 정규표현식을 매개 변수로 받는다.
```
String[] split = input.split(" ");
// 또는 String[] split = input.split("\\s");
```

### 문자열 인덱스 기준으로 자르기
* substring은 시작 인덱스부터 끝 인덱스까지 잘라준다. 
  * 끝 인덱스에 위치한 문자는 포함되지 않는다. 
* 끝 인덱스를 작성하지 않은 경우, 문자열의 끝까지 자른다. 
* 원본 문자열에는 영향을 주지 않는다.
```
String str = "Hello World!";
int startIndex = 0, endIndex = 5;
String result = str.substring(startIndex, endIndex);
System.out.println(str);
System.out.println(result);
```

### 문자열 뒤집기
* String을 charArray로 바꾸어 배열 역순으로 순회할 수 있다.
  * 아래의 코드는 result에 값을 더하는 과정에서 불필요한 String을 계속해서 만드는 단점이 있다.
  * 이러한 문제를 개선하기 위해 while(left < right)를 활용하여 왼쪽 끝의 값과 오른쪽 끝의 값을 바꾸어 나가는 방식을 적용하는 방법도 있다.
```
public static String reverseString(String current) {
    String result = "";
    int endIndex = current.length() - 1;
    char[] chars = current.toCharArray();

    for(int i = endIndex; i >= 0; i--)
        result += chars[i];

    return result;
}
```
* 또는 StringBuilder 클래스로 쉽게 구현할 수도 있다!
```
public static String reverseWithStringBuilder(String current) {
    return new StringBuilder(current).reverse().toString();
}
```
* 참고: String과 StringBuilder
  1. String은 불변 객체이므로, String을 더하거나 하는 변조 작업은 항상 새로운 String 인스턴스를 만든다.
  2. 반면 StringBuilder의 경우, toString을 통해 String 인스턴스를 생성하기 이전까지의 변조 작업이 새로운 객체를 만들어내지 않는다.
  * 즉, **두 클래스의 가장 큰 차이는 변조 과정에서 새로운 인스턴스가 만들어지는지의 여부**이다.
  * StringBuilder가 String보다 딱히 무겁지도 않으므로, 변조 작업이 많은 경우 StringBuilder를 활용하는 것이 바람직하다.

### 문자열에서 정규 표현식을 활용하여 영어 대문자 이외의 특수문자 제거하기
* 정규 표현식에서 부정 표현은 ^을 활용한다.
```
String str = target.replaceAll("[^A-Z]", "");
```

## 문자(char)
### 문자 읽어들이기
* 우선 문자열을 받아온 후, charAt 메소드로 인덱스를 활용한다
```
Scanner scanner = new Scanner(System.in);
char character = scanner.next().charAt(0);
```

### 문자를 정수로 변환
* int 형이 char 형보다 크므로 오토캐스팅된다.
```
int ascii = character;
```

### 문자의 대소 판별
* ASCII 코드를 사용할 수도 있지만, Character.isLowerCase 또는 isUpperCase를 활용하는 것이 가독성이 좋다.
  * ASCII 방식: 97 <= char <= 122 가 소문자 범위, 65 <= char <= 90이 대문자 범위
```
if(Character.isLowerCase(character))
System.out.println(character);
```

### 문자 배열을 문자열로 변환
* String 클래스의 정적 메소드인 valueOf()를 활용한다.
```
char[] charArray = { 'h', 'e', 'l', 'l', 'o', };
String string = String.valueOf(charArray);
System.out.println(string);
```

### 문자의 대소 판별하기
* ASCII 코드 범위를 활용할 수도 있지만, Character 클래스의 정적 메소드는 isAlphabetic은 더 좋은 가독성을 갖는다.
```
char character = 'c';
if(Character.isAlphabetic(character))
  System.out.println("Yes!");
```

### 문자가 숫자인지 판별하기
* 역시 Character의 정적 메소드인 isDigit을 활용하는 방식과, ASCII 코드를 활용하는 방식이 있다.
  * ASCII 범취: 48 <= character <= 57
```
if(Character.isDigit(character))
  System.out.println("digit!");
// 또는 메소드를 직접 구현
public static boolean isDigit(char character) {
  return character >= 48 && character <= 57;
}
```
## 숫자
### 문자열을 숫자로 변환
* Integer 클래스의 정적 메소드인 parseInt를 활용한다.
  * 이 경우, 0200은 200으로 변환 되는 식으로 앞의 0을 무시한다.
```
String string = "0200";
System.out.println(Integer.parseInt(string));
```