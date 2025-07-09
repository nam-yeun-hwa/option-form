


## React Hook Form을 사용하는 이유
React Hook Form은 폼 관리 라이브러리 중 가장 널리 사용되는 옵션 중 하나로, 개발자들과의 협업에서 가장 이점을 가지고 있다고 생각하여 사용하게 되었습니다.

- <b>최소 리렌더링</b> <br/>
  React Hook Form은 내부적으로 ref를 사용해 폼 상태를 관리하므로, 상태 변경 시 불필요한 리렌더링을 줄입니다. 이는 성능 최적화에 매우 유리합니다.
  
- <b>간단한 API</b> <br/> useForm, register, handleSubmit 같은 API는 직관적이고 사용이 쉬워, 복잡한 폼 로직을 간단히 처리할 수 있습니다.
  
- <b>유효성 검사 내장</b> <br/> Yup, Joi, Zod 같은 스키마 기반 유효성 검사 라이브러리와 쉽게 통합되며, HTML5의 기본 유효성 검사도 지원합니다.
  
- <b>커스터마이징 용이</b> <br/> 현재 코드처럼 FormInput 컴포넌트를 만들어 커스텀 스타일과 기능을 추가하기에 적합합니다.
  
- <b>작은 번들 사이즈</b> <br/> React Hook Form은 경량 라이브러리로, 번들 크기에 큰 영향을 주지 않습니다.
  
- 현재 코드에서 register와 <b>errors</b>를 활용해 입력 필드와 에러 메시지를 관리할수 있습니다.

## register


```
import { UseFormRegister, FieldValues } from "react-hook-form";

type RegisterFunction = UseFormRegister<FieldValues>;
````

- <b>UseFormRegister</b> : react-hook-form에서 제공하는 타입으로, register 함수의 시그니처를 정의합니다.
- <b>FieldValues</b> : 폼 필드의 값들을 나타내는 제네릭 타입입니다. 기본적으로 FieldValues는 모든 폼 필드를 포괄하지만, 특정 폼 필드의 타입을 명시하려면 제네릭으로 커스텀 타입을 전달할 수 있습니다.


```
register: (name: string, options?: RegisterOptions) => {
  onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur: (event: React.FocusEvent<HTMLInputElement>) => void;
  ref: (instance: HTMLInputElement | null) => void;
  name: string;
};
```

# 트러블슈팅 기록
## @iconify/react 이슈 
@iconify/react 라이브러리를 사용하여 하단 메뉴를 구성 하였는데 

```
import { Icon } from '@iconify/react';

function App() {
  return (
    <div>
      <Icon icon="mdi:home" />
      <Icon icon="fa-solid:coffee" />
      <Icon icon="tabler:heart" width="24" height="24" />
    </div>
  );
}
```
페이지에 진입할때마다 깜빡임 현상이 발생 하였다. 그리고 하여 다른 방법을 찾아서 기록 한다.</br>
<pre>
public
└── icons
    └── mdi-home.svg
</pre>
위와같은 형태로 svg 파일을 로드하여 사용하는 방법
   ```
    <Image src="/icons/mdi-home-active.svg" alt="Home Icon" width={30} height={30} priority />
   ```
## "Cannot find T" 에러 발생
상위 부모에서 타입을 내려받아서 사용하도록 제너릭으로 사용하도록 하는 과정에서 발생 하였다.
```

export interface PaymentInputProps<T extends FieldValues> {
  control: Control<T>;
  name: FieldPath<T>;
  errors: FieldErrors<T>;
  label: string;
  placeholder?: string;
  disabled?: boolean;
  inputMode?: "text" | "search" | "email" | "tel" | "url" | "none" | "numeric" | "decimal" | undefined;
  pattern?: string;
  noOutline?: boolean;
}


const FormInputPayment: React.FC<PaymentInputProps<T>(💡여기에서) > = ({
  control,
  name,
  errors,
  placeholder = "",
  disabled,
  pattern,
  label,
}) => {  
````

### 💡 오류가 난 문제의 코드 문제원인

```
const FormInputPayment: React.FC<PaymentInputProps<T>> = ({
  control,
  name,
  errors,
  placeholder = "",
  disabled,
  pattern,
  label,
}) => { ... }
```
기본적으로 "Cannot find T" 에러는 TypeScript에서 제네릭 타입 T가 정의되지 않았거나, T를 사용하는 컴포넌트에서 필요한 타입 정보를 제공하지 않아 발생하였다.<br/><br/>
T extends FieldValues는 react-hook-form의 FieldValues 타입을 확장한 제네릭 타입입니다. 하지만 T가 FormInputPayment 컴포넌트를 사용할 때 구체적으로 정의되지 않으면 TypeScript가 T를 추론하지 못해 에러가 발생했습니다.<br/><br/>
여기서는 React.FC<PaymentInputProps<T>>에서 T를 정의하지 않았기 때문에 TypeScript가 T를 찾을 수 없었습니다. React.FC는 제네릭 타입을 컴포넌트 선언 시 명시적으로 전달받아야 합니다(예: FormInputPayment<FormData>).


### 💡 해결

```
const FormInputPayment = <T extends FieldValues>({
  control,
  name,
  errors,
  placeholder = "",
  disabled,
  pattern,
  label,
}: PaymentInputProps<T>) => { ... }
```

이 방식은 컴포넌트 정의 자체에 <T extends FieldValues>를 포함해 제네릭 타입을 선언합니다. 이렇게 하면 TypeScript가 T를 함수 시그니처에서 직접 추론하거나 상위 컴포넌트에서 전달된 타입을 통해 해결할 수 있습니다.<br/><br/>
핵심 차이: 제네릭 선언을 React.FC 타입 지정에서 하는 대신, 함수 표현식의 파라미터 정의에 직접 포함시켰습니다. 이 방식은 react-hook-form 같은 제네릭-heavy 라이브러리에서 자주 사용됩니다.
