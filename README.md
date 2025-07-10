


## React Hook Form을 사용하는 이유
React Hook Form은 폼 관리 라이브러리 중 가장 널리 사용되는 옵션 중 하나로, 개발자들과의 협업에서 가장 이점을 가지고 있다고 생각하여 사용하게 되었습니다.

- <b>최소 리렌더링</b> <br/>
  React Hook Form은 내부적으로 ref를 사용해 폼 상태를 관리하므로, 상태 변경 시 불필요한 리렌더링을 줄입니다. 이는 성능 최적화에 매우 유리합니다.
  
- <b>간단한 API</b> <br/> useForm, register, handleSubmit 같은 API는 직관적이고 사용이 쉬워, 복잡한 폼 로직을 간단히 처리할 수 있습니다.
  
- <b>유효성 검사 내장</b> <br/> Yup, Joi, Zod 같은 스키마 기반 유효성 검사 라이브러리와 쉽게 통합되며, HTML5의 기본 유효성 검사도 지원합니다.
  
- <b>커스터마이징 용이</b> <br/> 현재 코드처럼 FormInput 컴포넌트를 만들어 커스텀 스타일과 기능을 추가하기에 적합합니다.
  
- <b>작은 번들 사이즈</b> <br/> React Hook Form은 경량 라이브러리로, 번들 크기에 큰 영향을 주지 않습니다.
  
- 현재 코드에서 register와 <b>errors</b>를 활용해 입력 필드와 에러 메시지를 관리할수 있습니다.

## 언제register를 사용할까?

register는 HTML input 요소에 직접 바인딩되며, react-hook-form이 내부적으로 DOM 이벤트를 감지해 값을 관리. <br/>
React 상태를 최소화하고, 네이티브 HTML input의 기본 동작에 의존하므로 렌더링 오버헤드가 적음.<br/>
간단한 폼 입력 처리 시 매우 가볍고 효율적.<br/>
코드가 간결함. register(name)을 input에 직접 적용하면 끝.<br/>
기본 HTML input 동작에 의존하므로 추가 로직이 필요 없는 경우 매우 직관적.<br/><br/>

```
<input {...register("name")} />
```

- 간단한 입력 필드(텍스트, 이메일, 비밀번호 등)를 처리할 때.
- HTML input 요소의 기본 동작으로 충분할 때.
- 포맷팅이나 커스텀 로직이 필요 없는 경우.
- 코드가 간결하고 빠르게 작성되어야 할 때.




## 언제 Controller를 사용해야 할까?



Controller는 react-hook-form에서 입력 필드의 동작을 세밀하게 제어할 때 사용됩니다. 


### 1. 커스텀 입력 처리 필요
상황 : 입력값을 포맷팅(예: 전화번호에 하이픈 추가)해야 할 때.
Controller를 사용해 전화번호 입력값을 포맷팅(하이픈 추가)하고, 내부적으로는 하이픈 없는 숫자만 저장. <br/>(register로는 이런 커스텀 포맷팅을 처리하기 어렵다.)
```
import { Controller, useForm } from "react-hook-form";
import classNames from "classnames";

interface FormData {
  phone: string;
}

const PhoneInput = () => {
  const { control, handleSubmit } = useForm<FormData>();

  // 전화번호 포맷팅 (예: 01012345678 -> 010-1234-5678)
  const formatPhoneNumber = (value: string) => {
    const cleaned = value.replace(/\D/g, "");
    const match = cleaned.match(/^(\d{0,3})(\d{0,4})(\d{0,4})$/);
    if (match) {
      return [match[1], match[2], match[3]].filter(Boolean).join("-");
    }
    return value;
  };

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="phone"
        control={control}
        rules={{ required: "전화번호를 입력하세요" }}
        render={({ field: { onChange, value, ...field }, fieldState: { error } }) => (
          <div>
            <input
              {...field}
              type="text"
              placeholder="010-1234-5678"
              value={formatPhoneNumber(value || "")}
              onChange={(e) => {
                const formatted = formatPhoneNumber(e.target.value);
                onChange(formatted.replace(/-/g, "")); // 하이픈 제거 후 저장
              }}
              className={classNames("w-full h-12 px-4 rounded-md border", {
                "border-red-500": error,
              })}
            />
            {error && <p className="text-red-500 text-sm">{error.message}</p>}
          </div>
        )}
      />
      <button type="submit">제출</button>
    </form>
  );
};

export default PhoneInput;
```

### 2. 비표준 입력 컴포넌트 사용
Material-UI의 TextField를 react-hook-form과 연동. <br/>
Material-UI의 TextField는 표준 HTML input이 아니므로 register를 직접 적용할 수 없다. 
```
import { Controller, useForm } from "react-hook-form";
import { TextField } from "@mui/material";

interface FormData {
  username: string;
}

const MaterialUIInput = () => {
  const { control, handleSubmit } = useForm<FormData>();

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="username"
        control={control}
        rules={{ required: "사용자 이름을 입력하세요" }}
        render={({ field, fieldState: { error } }) => (
          <TextField
            {...field}
            label="사용자 이름"
            variant="outlined"
            fullWidth
            error={!!error}
            helperText={error?.message}
          />
        )}
      />
      <button type="submit">제출</button>
    </form>
  );
};

export default MaterialUIInput;
```

### 복잡한 검증 로직
비동기 검증(예: 서버에서 사용자 이름 중복 확인).
Controller의 rules.validate를 사용해 비동기 검증 로직을 구현. (register로는 비동기 검증을 처리하기 복잡하다.)
```
import { Controller, useForm } from "react-hook-form";
import classNames from "classnames";

interface FormData {
  username: string;
}

const AsyncValidationInput = () => {
  const { control, handleSubmit } = useForm<FormData>();

  // 비동기 중복 확인 (서버 API 호출 시뮬레이션)
  const checkUsername = async (username: string) => {
    await new Promise((resolve) => setTimeout(resolve, 1000)); // 1초 지연
    if (username === "admin") return "이미 사용 중인 사용자 이름입니다";
    return true;
  };

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="username"
        control={control}
        rules={{
          required: "사용자 이름을 입력하세요",
          validate: async (value) => await checkUsername(value),
        }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <input
              {...field}
              type="text"
              placeholder="사용자 이름"
              className={classNames("w-full h-12 px-4 rounded-md border", {
                "border-red-500": error,
              })}
            />
            {error && <p className="text-red-500 text-sm">{error.message}</p>}
          </div>
        )}
      />
      <button type="submit">제출</button>
    </form>
  );
};

export default AsyncValidationInput;
```
### 상태 관리 통합
외부 상태(예: Redux, Zustand)와 입력값 동기화.
Controller를 사용해 react-hook-form의 상태와 외부 상태를 동기화. onChange를 커스텀해 두 상태를 모두 업데이트.
```
import { Controller, useForm } from "react-hook-form";
import { useState } from "react";
import classNames from "classnames";

interface FormData {
  email: string;
}

const StateSyncInput = () => {
  const { control, handleSubmit } = useForm<FormData>();
  const [externalState, setExternalState] = useState("");

  const onSubmit = (data: FormData) => {
    console.log(data, externalState);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="email"
        control={control}
        rules={{ required: "이메일을 입력하세요" }}
        render={({ field: { onChange, ...field }, fieldState: { error } }) => (
          <div>
            <input
              {...field}
              type="email"
              placeholder="이메일"
              onChange={(e) => {
                onChange(e); // react-hook-form 상태 업데이트
                setExternalState(e.target.value); // 외부 상태 업데이트
              }}
              className={classNames("w-full h-12 px-4 rounded-md border", {
                "border-red-500": error,
              })}
            />
            {error && <p className="text-red-500 text-sm">{error.message}</p>}
            <p>외부 상태: {externalState}</p>
          </div>
        )}
      />
      <button type="submit">제출</button>
    </form>
  );
};

export default StateSyncInput;
```


### 복합 UI
입력 필드에 라벨, 단위, 아이콘 등을 포함한 복합 UI (예: 금액 입력).
Controller를 사용해 금액 입력 필드에 라벨(‘금액’), 단위(‘원’), 쉼표 포맷팅을 포함한 복합 UI 구현. register로는 복합 UI와 포맷팅을 동시에 처리하기 어렵다.
```
import { Controller, useForm } from "react-hook-form";
import classNames from "classnames";

interface FormData {
  amount: number;
}

const ComplexUIInput = () => {
  const { control, handleSubmit } = useForm<FormData>();

  const formatNumberWithCommas = (value: string | number): string => {
    if (!value) return "";
    const cleanedValue = value.toString().replace(/,/g, "");
    return Number(cleanedValue).toLocaleString("ko-KR");
  };

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="amount"
        control={control}
        rules={{ required: "금액을 입력하세요", min: { value: 0, message: "0 이상이어야 합니다" } }}
        render={({ field: { onChange, value, ...field }, fieldState: { error } }) => (
          <div className="flex items-center border rounded-md bg-gray-100 p-2">
            <span className="text-sm text-gray-700 pr-2">금액</span>
            <input
              {...field}
              type="text"
              placeholder="0"
              inputMode="numeric"
              value={formatNumberWithCommas(value || "")}
              onChange={(e) => {
                const inputValue = e.target.value.replace(/,/g, "");
                onChange(inputValue ? Number(inputValue) : undefined);
              }}
              className={classNames("w-full h-10 text-right bg-transparent", {
                "border-red-500": error,
              })}
            />
            <span className="text-sm text-gray-500 pl-2">원</span>
            {error && <p className="text-red-500 text-sm mt-1">{error.message}</p>}
          </div>
        )}
      />
      <button type="submit">제출</button>
    </form>
  );
};

export default ComplexUIInput;
```

- Controller는 React 컴포넌트로, render prop을 통해 입력 필드를 렌더링하며 react-hook-form과 입력값을 연결.<br/>
- render 함수가 매 렌더링마다 호출되므로, 렌더링 오버헤드가 register보다 약간 더 큼.<br/>
특히 커스텀 로직(포맷팅, 상태 동기화 등)이 추가되면 약간의 계산 비용이 증가.<br/>
하지만 react-hook-form은 최적화가 잘 되어 있어, 일반적인 사용 사례에서는 이 오버헤드가 눈에 띄게 크지 않음.<br/>
render prop과 커스텀 로직(onChange, value 처리 등)을 작성해야 하므로 코드가 더 장황해질 수 있음.<br/>
특히 포맷팅, 비표준 컴포넌트 연동, 복잡한 검증 등이 추가되면 코드량 증가.<br/>
대규모 폼(100개 이상 입력 필드)에서는 Controller의 렌더링 오버헤드가 누적될 수 있으니, 불필요한 Controller 사용은 피하고 register를 우선 고려.<br/>

```
<Controller
  name="name"
  control={control}
  render={({ field }) => <input {...field} />}
/>
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

#### register의 사용 타입스크립트 기본
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
