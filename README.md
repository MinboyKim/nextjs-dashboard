# Chapter 1

## Creating a new project

```bash
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

## Folder structure

- `/app` : routes, components, 앱의 로직 등 포함. 대부분의 시간을 여기서 보낼 것
- `/app/lib` : 재사용 가능한 함수들 - 유틸성 함수, 데이터 fetching 함수 등
- `/app/ui` : UI components - 카드, 테이블, 폼 등등
- `/public` : 이미지와 같은 정적 파일들
- `/scripts` : 차후에 db를 만들 때 사용할 스크립트

## Running the development server

```bash
npm run dev
```

# Chapter 2

## Global styles

`/app/ui` 폴더 내부의 `global.css` 파일에서 모든 routes에 적용될 CSS 규칙 - CSS reset, 링크스타일 등을 정의

어느 컴포넌트에서도 import 하여 사용할 수 있지만, 최상위 컴포넌트에서 적용하는 것이 좋은 관습이다. Next.js에서는 이를 `root layout` 이라한다.

`/app/layout.tsx` 에서 import 하여 글로벌 스타일을 적용하자.

## Tailwind

클래스명을 통해 스타일링을 하는 Tailwind 프레임워크를 사용할 것을 권장하고 있다. `create-next-app` 을 이용해 프로젝트를 셋업할 때 Tailwind를 사용할 것인지 묻는데, 이때 `yes`를 선택하면 자동으로 필요한 패키지와 설정을 완료해준다.

일전에 Tailwind를 사용해본 경험이 있고, 빠른 개발 프로세스라는 장점에 정말 좋게 생각하고 있던 프레임워크여서 굉장히 반가웠다.

[Installation - Tailwind CSS](https://tailwindcss.com/docs/installation)

Tailwind의 공식문서도 굉장히 친절하게 설명하고 있고, 원하는 스타일의 클래스명도 검색을 통해 쉽게 찾을 수 있다.

## Using the `clsx` library to toggle class names

조건적으로 클래스명을 주기 위해 `clsx` 라이브러리를 사용할 것을 추천하고 있다.

```tsx
import clsx from 'clsx';

export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

이런식으로 클래스명에 `clsx({”class-name” : boolean})` 와 같이 사용하여 조건부 클래스명을 전달할 수 있다.

# Chapter 3

## Why optimize fonts?

폰트는 웹 디자인에서 큰 역할을 담당하지만, 커스텀 폰트를 이용하는 것은 폰트를 `fetch`하고 `load`하는 과정이 필요하다면 퍼포먼스에 영향을 줄 수도 있다.

폰트가 로드되면서 텍스트 크기, 여백, 레이아웃 등에 영향을 미칠 수 있고, 이는 나쁜 사용자 경험을 가져올 수 있다. 이러한 사용자 경험 퍼포먼스를 측정하는 `[Cumulative Layout Shift](https://web.dev/articles/cls?hl=ko)` 라는 구글에서 사용되는 척도도 있다.

Next.js는 `next/font` 모듈을 이용하면 자동으로 폰트를 최적화 해준다. 폰트파일을 빌드 타임에 다운로드 받고 정적파일들과 함께 호스팅 해주는데, 이를 통해 사용자가 어플리케이션에 접근했을 때 추가적인 폰트 요청을 하지 않아도 되게 함으로써 사용자 경험 퍼포먼스 문제를 해결한다.

```tsx
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

`fonts.ts` 파일에서 폰트를 정의해주고,

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

원하는 곳에서 위처럼 사용하면 된다.

## Why optimize images?

일반적인 HTML의 `img` 태그를 사용하면 다음을 수동적으로 처리해주어야한다.

- 다양한 스크린 사이즈에서의 반응형 보장
- 다양한 장치에서의 이미지 크기 명시
- 이미지 로드 시 레이아웃 밀림 방지
- 사용자의 시점 밖의 이미지 지연 로딩

이러한 이미지 최적화를 하나하나 구현하는 대신, `next/image` 컴포넌트를 활용하면 쉽게 처리할 수 있다.

## The `<Image>` component

`<Image>` 컴포넌트는 HTML의 `img` 태그를 확장한 것으로, 상기한 이미지 최적화를 자동으로 처리해준다.

```tsx
import Image from 'next/image';

...
	<Image
    src="/hero-desktop.png"
    width={1000}
    height={760}
    className="hidden md:block"
    alt="Screenshots of the dashboard project showing desktop version"
  />
  <Image
    src="/hero-mobile.png"
    width={560}
    height={620}
    className="block md:hidden"
    alt="Screenshots of the dashboard project showing mobile version"
  />
...
```

위와 같이 사용하면 된다. `md:block` 클래스명은 다음과 같은 CSS 효과를 준다.

```css
@media (min-width: 768px) {
  .md\:block {
    display: block;
  }
}
```

이를 활용해 `hidden md:block` 과 `block md:hidden` 클래스명을 통해 데스크탑과 모바일에서의 조건을 걸어줄 수 있다.

# Chapter 4

## Nested routing

Next.js는 폴더들이 중첩 라우팅을 하는데 사용되는 `파일 시스템 기반 라우팅`을 사용한다. 각각의 폴더가 **route segment**가 되어 **URL segment**에 대응된다.

![1](https://github.com/MinboyKim/monorepo_test/assets/35567292/fc393fa8-2de5-4696-96bb-1be13f7d7121)

`layout.tsx` 와 `page.tsx` 라는 특별한 파일을 통해 각각의 라우트에 해당하는 개별 UI를 만들 수 있다.

![2](https://github.com/MinboyKim/monorepo_test/assets/35567292/9a0243a6-6c9f-4ea5-b669-cc485a7d76ec)

폴더명을 통해 라우트를 정의하고, `page.tsx` 파일에서 export하는 리액트 컴포넌트를 통해 해당 라우트에 접근했을 때 보여질 UI를 정의한다. 예를 들어 `/dashboard` 라우트에 접근하면 `app/dashboard/page.tsx` 에 정의되어있는 리액트 컴포넌트가 렌더링 될 것이다.

![3](https://github.com/MinboyKim/monorepo_test/assets/35567292/7f379624-9d7d-4e3d-bb0a-8082aa6c02c4)

위와 같이 중첩 라우팅을 구성해주자.

## Creating the dashboard layout

여러 라우트에서 공통된 UI - 예를들면 Nav, Footer 등과 같은 - 가 필요하다면 `layout.tsx` 파일을 이용하면된다.

예를들어 `/dashboard/customers` 라우트와 `/dashboard/invoices` 라우트에서 공통적으로 사용되는 `SideNav` 가 필요하다면 `/dashboard` 폴더에 `layout.tsx` 파일을 만들어주면 된다.

```tsx
import SideNav from '@/app/ui/dashboard/sidenav';

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
  );
}
```

이렇게 `layout.tsx` 를 활용하면 여러 라우트에서 공통적으로 활용하는 UI를 정의해줄 수 있다. `<Layout />` 컴포넌트는 `children` prop을 받는데, 여기에 다른 레이아웃들과 페이지들이 전달된다.

![4](https://github.com/MinboyKim/monorepo_test/assets/35567292/9c81f125-d287-4e56-bc7e-4bf7bf1617d3)

이렇게 Next.js의 레이아웃을 활용하면 좋은 점으로 부분 렌더링을 지원해준다. 즉 라우팅 변경 시 `children` 부분만 업데이트 되고 레이아웃은 재렌더링 하지 않음으로써 자원을 절약할 수 있다.

![5](https://github.com/MinboyKim/monorepo_test/assets/35567292/3600e39b-37d4-4dd0-a187-fef30c1a942e)

## Root layout

```tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

`/app/layout.tsx` 파일은 반드시 존재해야하는 Root layout이다. 이곳에 추가하는 모든 UI는 모든 페이지에 적용될 것이다. 여기서 `<html>` 태그와 `<body>` 태그를 수정할 수 있고, 메타데이터도 추가할 수 있다.
