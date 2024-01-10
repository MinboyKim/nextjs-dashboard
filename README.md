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

# Chapter 5

## Why optimize navigation?

원래 링크간의 연결을 구현하기 위해서 전통적으로는 HTML의 `<a>` 태그를 활용하지만, 이를 이용하면 전체 페이지가 리프레쉬 되는 문제가 발생한다. 이를 해결하기 위한 방법을 알아보자.

## The `<Link>` component

Next.js에서는 페이지 간의 링크를 연결하기 위해 `<Link />` 컴포넌트를 사용한다. `<Link>` 컴포넌트는 자바스크립트를 통해 클라언트 측 네비게이션을 가능하게 해준다.

```tsx
import Link from 'next/link';

...
	<Link
    key={link.name}
    href={link.href}
    className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
  >
    <LinkIcon className="w-6" />
    <p className="hidden md:block">{link.name}</p>
   </Link>
...
```

위와 같이 `<a>` 태그 대신에 `next/link` 에서 `<Link>` 컴포넌트를 import 해와 사용하면 된다.

이렇게 `<Link>` 컴포넌트를 사용하면 페이지 이동 시에 리프레쉬 되지않고 렌더링이 되는 모습을 확인할 수 있다.

## Automatic code-splitting and prefetching

네비게이션 경험 향상을 위해, Next.js는 자동적으로 우리의 코드를 라우트에 따라 **분할한다**. 이는 첫 initial load에 모든 코드를 브라우저가 로드해오는 전통적인 React SPA 방식과는 차별화된다.

코드를 라우트에 따라 분할하는 것으로 페이지는 독립적이게 되고, 특정 페이지가 에러가 발생하더라도 전체 앱이 crash 되지 않고 여전히 동작할 수 있다.

추가로, 브라우저의 화면에 `<Link>` 컴포넌트가 등장하면, Next.js는 자동적으로 백그라운드에서 해당 링크에 연결된 라우트의 코드를 prefetch, 미리 가져온다. 유저가 해당 링크를 클릭하면, 이미 준비된 코드를 보여줌으로써 페이지 전환이 거의 즉각적으로 보여지게 되는 것이다.

## Pattern: Showing active links

현재 사용자가 있는 페이지를 표현하기 위한 UI 패턴으로, 액티브 링크를 보여줄 수 있다. 이를 위해서는 현재 유저가 있는 URL path를 가져올 필요가 있는데, Next.js에서는 `usePathname()` 훅을 통해 이를 가져올 수 있다.

`usePathname()` 은 훅이기 때문에 사용하는 곳을 클라이언트 컴포넌트로 선언해주어야한다. 해당 컴포넌트의 최상단에 `"use client"` 를 선언해주고, `next/navigation` 에서 `usePathname()` 훅을 import 해오자.

```tsx
'use client';

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import clsx from 'clsx';

// ...

export default function NavLinks() {
  const pathname = usePathname();

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

이렇게 clsx를 함께 활용해 사용자의 현재 URL과 일치하는 링크의 스타일을 다르게 적용해주는 것으로 액티브 링크를 구현해줄 수 있다.

# Chapter 6

해당 챕터에서는 Github 레포지토리를 생성하고, Vercel과 연동하여 배포, Postgres database를 생성하여 연결을 진행한다. `/scripts` 폴더에 존재하는 `seed.js` 파일의 스크립트를 통해 생성한 데이터베이스에 테이블들을 만들고, `/app/lib` 폴더의 `placeholder-data.js` 에 존재하는 목업 데이터들을 넣는 것 까지 진행한다.

Vercel을 이번기회에 처음 적용해보게되었는데, 굉장히 간편하고 빠르게 사용할 수 있는 것 같다. DB 생성과 연결까지 간단하게 처리할 수 있으니 혼자 프로젝트 진행할 때 써먹어도 참 좋을 것 같다. 👍

# Chapter 7

## Using Server Components to fetch data

Next.js에서는 데이터 페칭을 위해 디폴트로 React Server Components를 이용한다. 서버 컴포넌트를 활용해 데이터에 접근하는 것은 다음과 같은 장점이있다.

- 서버 컴포넌트는 promise를 지원하기 때문에, 데이터 페칭과 같은 비동기 작업에 간단한 해결책이다. `useEffect`, `useState` 와 같은 훅이나 데이터 페칭 라이브러리 없이 `async/await` 만으로도 데이터 페칭을 해결할 수 있다.
- 서버 컴포넌트는 서버에서 실행되기 때문에,데이터 접근, 로직과 같은 비싼 작업을 서버에서 처리하고 결과만 클라이언트에 보낼 수 있다.
- 상기한대로 서버 컴포넌트는 서버에서 실행되기 때문에 추가적인 API 레이어 없이도 데이터베이스에 직접 쿼리를 보낼 수 있다.

## Using SQL

해당 프로젝트에서는 Vercel Postgres SDK와 SQL을 이용한 데이터 베이스 쿼리를 작성한다. SQL을 이용하는 이유는 다음과 같다.

- SQL은 RDB에 쿼리를 보내는 산업 표준이다. (ORM도 내부적으로 SQL을 생성한다.)
- SQL에 대한 기본적인 이해는 RDB의 기반을 이해하고, 다른 도구들을 사용할 수 있는 지식을 제공한다.
- SQL은 특정 데이터에 접근하고 조작하는데 다재다능하다.
- Vercel Postgres SDK는 SQL injection에 대한 보호를 제공한다.

`/app/lib/data.ts` 에서 데이터베이스에 쿼리를 보내는 모습을 확인할 수 있다.

```tsx
import { sql } from '@vercel/postgres';

...
	const data = await sql<Revenue>`SELECT * FROM revenue`;
...
```

이런식으로 데이터베이스에 쿼리를 보낼 수 있다.

## Fetching data for the dashboard overview page

```tsx
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import { fetchCardData, fetchLatestInvoices, fetchRevenue } from '../lib/data';

export default async function Page() {
  const revenue = await fetchRevenue();
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfCustomers,
    numberOfInvoices,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Card title="Collected" value={totalPaidInvoices} type="collected" />
        <Card title="Pending" value={totalPendingInvoices} type="pending" />
        <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
        <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        />
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <RevenueChart revenue={revenue} />
        <LatestInvoices latestInvoices={latestInvoices} />
      </div>
    </main>
  );
}
```

위 코드를 보면

- Page 컴포넌트는 **async** 컴포넌트이다. 이를 통해 데이터 페칭 시 `await` 을 이용할 수 있다.
- 데이터를 필요로하는 `<Card>`, `<RevenueChart>`, `<LatestInvoices>` 컴포넌트들이 있다.

해당 컴포넌트들에 필요한 데이터를 페칭해오기 위해 사용한 함수들을 확인해보면 위에서 말한 sql을 이용해 데이터베이스에 쿼리를 날려 데이터를 가져오는 모습을 확인할 수 있다.

하지만 짚고 넘어가야할 문제가 두 가지 존재한다.

1. 데이터 요청은 의도적이지 않게 서로를 blocking 할 수 있다. 이를 **request waterfall**이라고 한다.
2. 디폴트로, Next.js는 성능 향상을 위해 라우트들을 **사전 렌더링**하고, 이를 **Static Rendering**이라고 한다. 이때문에 만약 데이터가 바뀌어도 즉각 반영되지 않을 수 있다.

하나씩 짚어보자.

## What are request waterfalls?

“waterfall” (폭포)는 일련의 네트워크 요청이 다른 요청의 완료에 의존하고 있는 상태를 말한다. 데이터 접근의 경우에, 각각의 요청이 이전 요청들이 끝나야지만 시작되는 경우를 말한다.

![6](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/0a59f54d-32cd-4861-8870-6daa9618c789)

```tsx
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

예를들어 위의 경우에 `fetchLatestInvoices()` 함수가 실행되기 위해서는 `fetchRevenue()` 함수가 끝나야만 한다.

이러한 패턴은 필연적으로 좋지 않다. 다음 요청을 보내기 전에 특정 조건을 만족시켜야함을 확인해야한다거나 하는 경우에는 이러한 “waterfall”이 필요할 수도 있다. 예를 들어 유저의 ID와 프로필 정보에 접근할 때, ID를 먼저 받고 그 뒤에 해당 ID를 통해 요청을 보내야 하면 이전 요청인 ID 요청을 끝낸 뒤에 프로필 정보 요청을 보내야한다.

하지만 이러한 행동은 때때로 비의도적이고 성능에 영향을 끼칠 수도 있다.

## Parallel data fetching

이러한 waterfall을 해결하기 위한 흔한 방법으로는, 모든 데이터 요청을 병렬적으로 동시에 보내는 방법이 있다.

자바스크립트에서는 `Promise.all()` 또는 `Promise.allSettled()` 함수를 통해 모든 프로메스를 시작할 수 있다. 예를들어 `data.ts` 의 `fetchCardDatat()` 함수를 확인해보면

```tsx
export async function fetchCardData() {
  try {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
    // ...
  }
}
```

위와 같이 `Promise.all()` 함수를 이용하고 있다.

이러한 패턴을 이용함으로써

- 모든 데이터 접근을 동시에 시작하여 성능 향상을 얻을 수 있다.
- 네이티브 자바스크립트 패턴이므로 어떤 라이브러리나 프레임워크에서도 적용할 수 있다.

하지만, 이 방법 또한 한 가지 단점이 존재한다. 만약 한 데이터 요청이 다른 모든 요청들에 비해 매우 느리다면 어떻게 될까?

# Chapter 8

## What is Static Rendering?

Static Rendering, 정적 렌더링을 통해 데이터 접근과 렌더링은 서버에서 빌드 타임, 즉 배포시에 일어나거나 데이터 재평가시에 발생한다. 해당 결과는 [Content Delivery Network (CDN)](https://developer.mozilla.org/ko/docs/Glossary/CDN)에 분산되어 캐시될 수 있다.

![7](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/03ca3b1a-606a-4710-b0fc-6418c39fce22)

유저가 앱에 접근하면, 캐시된 결과가 제공된다. 이러한 정적 렌더링에는 다음과 같은 장점들이 있다.

- 빠른 웹사이트 - 사전 렌더링 된 컨텐츠를 캐시할 수 있고, 유저들은 웹사이트의 컨텐츠에 빠르고 확실하게 접근할 수 있다.
- 서버 부하 감소 - 컨텐츠들이 캐시되므로, 서버는 모든 각각의 유저들의 요청마다 매번 동적으로 컨텐츠를 생성할 필요가 없다.
- SEO - 사전 렌더링된 컨텐츠는 컨텐츠들이 사전 접근이 용이하기 때문에 검색 엔진 크롤러들이 색인을 생성하기에 더 쉽다. 이는 검색 엔진 순위 향상을 가져올 수 있다.

정적 렌더링은 블로그 게시글이나, 제품 페이지와 같은 데이터가 없거나, 유저들 간에 공유하는 데이터를 가진 UI에 매우 유용하다.

하지만 각각의 유저마다 다른 정보를 가진 개인화된 데이터나 자주 업데이트가 되는 유저 프로필, 대쉬보드 등과 같은 경우에는 좋은 선택이 아닐 수 있다. 이럴 때는 반대로 dynamic rendering, 동적 렌더링을 이용해야한다.

## What is Dynamic Rendering?

동적 렌더링을 통해 컨텐츠는 서버에서 각각의 유저가 보낸 요청에 따라 리퀘스트 타임에 렌더링된다. (유저 방문 등) 동적 렌더링의 장점은 다음과 같다.

- 실시간 데이터 - 동적 렌더링은 실시간 또는 자주 업데이트 된 데이터가 표시될 수 있게 한다. 이는 데이터가 자주 변경되는 경우에 적합하다.
- 유저 개인 컨텐츠 - 대쉬보드나 유저 프로필처럼 개인화된 데이터를 제공하고 상호작용하기에 적합하다.
- 리퀘스트 타임 정보 - 동적 렌더링은 리퀘스트 타임에만 알 수 있는 정보에 접근할 수 있게 해준다. 예를 들면 쿠키나 URL 서치 파라미터와 같은 정보들은 리퀘스트 타임에만 알 수 있고, 동적 렌더링을 이용해야한다.

## Making the dashboard dynamic

`@vercel/postgres` 는 캐시 행동을 디폴트로 정의하지 않는다. 따라서 정적이든, 동적이든 원하는 행동을 설정하면 된다.

Next.js API 중 `unstable_noStore` 를 이용해 동적 렌더링을 설정할 수 있다.

```tsx
import { unstable_noStore as noStore } from 'next/cache';

export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();

  // ...
}
```

위와 같이 사용하면 된다.

## Simulating a Slow Data Fetch

Promise와 setTimeout을 통해 느린 데이터 요청을 시뮬레이션해보자. 데이터에 접근하는 동안 전체 페이지가 블로킹 되는 결과를 확인할 수 있다.

동적 렌더링을 이용하면 우리의 앱은 가장 느린 데이터 페칭만큼만 빠를 수 있다!

# Chapter 9

이전 챕터에서, 대쉬보드 페이지를 동적으로 만들었지만 느린 데이터 페칭이 전체 앱의 성능에 영향을 미칠 수 있다는 것을 확인했다. 느린 데이터 요청에 대해 유저 경험을 향상시키는 방법을 알아보자.

## What is streaming?

스트리밍은 한 라우트를 더 작은 여러개의 “chunks” 들로 쪼개어 서버에서 준비되는 대로 점진적으로 불러오는 데이터 교환 기술이다.

![8](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/9f22df73-fdcd-4a04-ae3c-825eb9339618)

스트리밍을 통해 느린 데이터 요청이 전체 페이지를 블로킹하는 것을 방지할 수 있다.

![9](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/9d55d5a3-c0c8-49b3-9067-82dc40a9f3cf)

리액트의 컴포넌트 모델은 스트리밍과 잘 맞는데, 각각의 컴포넌트를 chunk로 생각할 수 있기 때문이다.

Next.js에서 스트리밍을 구현하는 방법은 두가지가 존재한다.

1. 페이지 레벨에서, `loading.tsx` 파일을 통해 구현
2. 구체적인 컴포넌트 레벨에서, `<Suspense>` 컴포넌트를 통해 구현

## Streaming a whole page with `loading.tsx`

`/app/dashboard` 폴더에 `loading.tsx` 파일을 새로 생성하자.

```tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

![10](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/9b1f027f-006a-4fbf-8109-4eab40567249)

1. `loading.tsx` 파일은 Suspense를 이용해 만들어진 특별한 Next.js 파일이다. 이를 통해 페이지가 컨텐츠를 로딩하는 동안 보여줄 fallback UI를 정의할 수 있다.
2. `<Sidebar>` 는 정적이므로 즉시 보여진다. 유저는 동적 컨텐츠들이 로딩되는 동안에도 `<Sidebar>` 와 상호작용할 수 있다.
3. 유저는 다른 경로로 이동하기 전에 페이지 로딩이 완료되는 것을 기다릴 필요가 없고, 이것을 interruptable navigation이라고 한다.

더 좋은 유저 경험을 위해 “Loading…” 텍스트가 아닌 스켈레톤 UI를 적용하면 좋다.

## Fixing the loading skeleton bug with route groups

현재 `loading.tsx` 파일의 레벨이 `/invoices/page.tsx` 와 `/customers/page.tsx` 파일보다 더 높기 때문에, 두 페이지에도 스켈레톤 UI가 적용되고 있는 버그가 존재한다.

이를 **Route Groups**를 통해 해결할 수 있다. dashboard 폴더에 `/(overview)` 폴더를 만들어 `loading.tsx` 와 `page.tsx` 를 이동시키자.

![11](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/37d2ec1c-749a-423d-8dc8-b723f84b79d7)

이제 `loading.tsx` 파일은 대쉬보드 오버뷰 페이지에만 적용될 것이다.

Route groups는 파일들을 URL path 구조에 영향을 끼치지 않으면서도 논리적인 그룹으로 분리할 수 있게 해준다. `()` 와 같이 소괄호로 감싼 폴더명으로 새로운 폴더를 만들면, 해당 이름은 URL path에 포함되지 않는다. 따라서 `/dashboard/(overview)/page.tsx` 는 그냥 `/dashboard` 가 되는 것이다.

여기서는 `loading.tsx` 파일이 대쉬보드 오버뷰 페이지에만 적용되게 하기 위해 라우트 그룹을 사용했지만, 더 큰 앱에서는 앱을 섹션별로 분류하기 위해 사용할 수도 있다.

## Streaming a component

지금까지는, 전체 페이지를 스트리밍했지만 이제 리액트의 Suspense를 활용해 더 잘게 쪼개어 보자.

Suspense는 특정 조건, 예를 들면 데이터가 로딩될 때까지 렌더링을 미룰 수 있게 해준다. 동적 컴포넌트를 Suspense로 감싸고 fallback 컴포넌트를 전달해주면 동적 컴포넌트가 로드될 때까지 fallback 컴포넌트가 보여질 것이다.

일전의 `fetchRevenue()` 와 같이 전체 페이지의 성능을 느리게하는 느린 데이터 요청이 전체 페이지를 블로킹하는 대신에, Suspense를 이용하여 해당 컴포넌트의 렌더링만 미루고 페이지의 나머지 UI들은 보일 수 있도록 만들 수 있다.

그렇게 하기 위해, 데이터 페칭을 해당 컴포넌트 내부로 옮겨주어야한다.

```tsx
...
<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
...
```

이렇게 Suspense로 감싸주고 fallback에 fallback UI를 전달해주면 된다.

![12](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/79887279-4923-4884-b046-d34ac4036bda)

이제 느린 데이터 요청을 하는 해당 부분만 로딩이 되고 나머지 부분들은 즉시 보여지는 것을 확인할 수 있다.

## Grouping component

네 개의 카드 컴포넌트로 이루어진 부분도 각각의 Card 컴포넌트에서 해당하는 데이터들을 페칭하게하고 Suspense를 통해 fallback UI를 보여줄 수 있을 것이다. 하지만 이렇게 너무 세분화하는 것은 오히려 유저에게 정신없어보이는 안좋은 경험을 줄 수 있다.

따라서 wrapper 컴포넌트로 카드들을 감싸 그룹화를 통해 한 번에 처리하는 것이 좋을 수 있다.

```tsx
export default async function CardWrapper() {
  const {
    numberOfCustomers,
    numberOfInvoices,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
  return (
    <>
      <Card title="Collected" value={totalPaidInvoices} type="collected" />
      <Card title="Pending" value={totalPendingInvoices} type="pending" />
      <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
      <Card
        title="Total Customers"
        value={numberOfCustomers}
        type="customers"
      />
    </>
  );
}
```

```tsx
...
<Suspense fallback={<CardsSkeleton />}>
  <CardWrapper />
</Suspense>
...
```

이렇게 wrapper 컴포넌트를 통해 그룹화를 하면 네개의 카드가 모두 데이터 페칭을 완료했을 때 동시에 카드들이 유저에게 보여지도록 할 수 있다.

## Deciding where to place your Suspense boundaries

Suspense 바운더리를 어디에 위치시킬 것인지는 다음과 같은 요소들에 의해 정해진다.

1. 페이지가 스트리밍 될 때 사용자가 어떻게 경험하게 하고 싶어하는지
2. 어떤 컨텐츠가 더 주요한지
3. 컴포넌트가 데이터 페칭에 의존하고 있는 경우

정답은 정해진 것이 없다.

- `loading.tsx` 를 통해 전체 페이지를 스트리밍 할 수도 있다. 하지만 이는 한 컴포넌트가 긴 데이터 요청을 하는 경우 느린 로딩을 가져올 수도 있다.
- 모든 각각의 컴포넌트를 스트리밍 할 수도 있다. 하지만 이는 UI가 전부 따로노는 듯한 경험을 가져올 수 있다.
- 페이지를 섹션들로 구분해 스트리밍을 할 수도 있다. 하지만 이는 wrapper 컴포넌트를 만들어야한다.

Suspense 바운더리를 어디에 위치시킬 지는 어떤 앱인지에 따라 다 다르다. 일반적으로 데이터 페칭은 컴포넌트로 내려보내고 Suspense로 감싸주는 것이 좋은 관습이다. 하지만 섹션별로, 또는 페이지별로 스트리밍하는 것은 틀린 것이 아니다.

많은 경험을 해보는 것이 중요하다.

# Chapter 10

## Combining Static and Dynamic Content

지금까지의 웹 페이지들은 어떤 특정 라우트 내에서 `noStore` 나 `cookies()` 와 같은 동적 함수들을 호출하면 라우트 전체가 동적으로 바뀐다.

하지만 대부분의 라우트들은 전체 페이지가 정적이거나 동적 하나만으로 결정되지 않고, 어느 부분은 정적이고 어느 부분은 동적이다. 예를들면 소셜 미디어 피드를 생각해보자. 포스트들은 정적이지만, 포스트들의 좋아요는 동적일 것이다. 전자 상거래 사이트를 생각해보면 제품 설명은 정적이지만, 유저 카트는 동적일 것이다.

## What is Partial Prerendering?

Next.js 14에 새로 추가된 기능인 Partial Prerendering을 통해 페이지 내의 정적인 부분과 동적인 부분들을 분리해 렌더링할 수 있게 되었다.

![13](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/25124f10-9c63-4f4f-8117-8ff83b5443b6)

만약 사용자가 특정 라우트에 방문하면,

- 정적인 부분들이 **먼저 제공**되고, 덕분에 초기 로딩이 빨라진다.
- 비동기적으로 로드될 동적인 컨텐츠들은 구멍으로 남아있다.
- 비동기 구멍들이 **병렬적으로** 로드됨에 따라 전체 페이지 로드 타임을 줄인다.

이 방식은 라우트 전체가 정적이거나 동적인 요즈음의 웹 페이지들과는 달리 한 라우트내에 정적인 부분들을 먼저 준비주고, 동적인 부분들을 병렬적으로 처리해준다. 결국 로드 타임을 줄이고 사용자 경험을 개선할 수 있으며, Next.js 팀은 이것이 웹 어플리케이션의 디폴트 렌더링 모델이 될 잠재력이 있다고 생각하고 있다고 한다.

## How does Partial Prerendering work?

Partial Prerendering은 리액트의 Concurrent API들과 Suspense를 이용한다.

초기 정적파일에 fallback이 다른 정적 컨텐츠들과 함께 삽입되고, 빌드타임 또는 재평가되는 동안 라우트의 정적인 부분들은 prerendered - 사전 렌더링 된다. 나머지 부분(동적인 부분들)은 라우트의 유저 요청이 올 때까지 지연된다.

컴포넌트를 Suspense로 감싸면 컴포넌트가 동적으로 변하는게 아닌, Suspense를 라우트의 정적과 동적 경계로 활용할 수 있다는 것에 주목하자. (컴포넌트를 동적으로 바꾸려면 unstable_noStore를 사용했던 것을 떠올리자.)

Partial Prerendering의 놀라운 점은 따로 코드를 변경할 필요가 없다는 점이다. 앞서 진행했던 것처럼 라우트의 동적인 부분을 Suspense로 감싸주기만 하면 Next.js는 라우트의 어떤 부분이 정적이고 동적인지 알아서 판단할 것이다.

## Summary

지금까지 데이터 페칭을 위해 진행한 최적화를 복습해보자.

1. 서버와 DB간의 레이턴시 감소를 위해 DB를 같은 리전에 생성
2. 데이터 페칭을 리액트 서버 컴포넌트에서 진행함으로써 높은 비용의 데이터 페칭과 로직을 서버단으로 옮기고 클라이언트 사이드의 JS 번들 사이즈를 줄일 수 있었다. 또한 이를 통해 DB Secrete을 클라이언트에 노출하는 위험을 없앨 수 있었다.
3. SQL을 사용하여 필요한 데이터만 가져오므로 각 요청에 대해 전송되는 데이터의 양과 메모리 내 데이터를 변환하는 데 필요한 JavaScript의 양을 줄였다.
4. 필요한 곳에서 JS를 이용해 데이터 페칭을 병렬화 했다.
5. 느린 데이터 요청이 전체 페이지를 블로킹하는 것을 예방하기 위해 스트리밍을 구현했고, 이를 통해 유저가 모든 페이지가 로드되기 전에도 UI와 상호작용할 수 있게 해주었다.
6. 데이터 페칭 코드를 해당 데이터를 필요로하는 컴포넌트 내부로 옮기고, 이를 통해 라우트의 각 부분들을 분리해 Partial Prerendering을 위해 동적으로 준비될 수 있게 해주었다.

# Chapter 11

## Why use URL search params?

- 북마크 및 공유 가능한 URL : 서치 파라미터가 URL 안에 있기 때문에, 유저는 서치 쿼리나 필터를 포함한 어플리케이션의 현재 상태를 북마크해 향후에 재방문 또는 공유할 수 있다.
- 서버 사이드 렌더링과 초기 로드 : URL 파라미터들은 초기 로드시에 즉시 서버로 전달되어 서버 렌더링을 쉽게 해준다.
- 분석과 추적 : 서치쿼리들과 필터들을 URL에 직접 갖고있는것은 추가적인 클라이언트 로직 없이 유저의 행동을 추적하기 쉽게 해준다.

## Adding the search functionality

- `useSearchParams` - 현재 URL의 파라미터들에 접근할 수 있게 해준다. 예를들어 `/dashboard/invoices?page=1&query=pending` 과 같은 URL의 서치 파라미터는 `{page: '1', query: 'pending' }` 과 같은 모습이다.
- `usePathname` - 현재 URL의 pathname을 읽을 수 있게 해준다. 예를들어 `/dashboard/invoices` 라우터에서 `usePathname` 은 `'/dashboard/invoice'` 를 리턴 할 것이다.
- `useRouter` - 라우트 간의 이동을 가능하게 해준다.

유저의 검색을 구현하는 단계는 다음과 같다.

1. 유저의 입력을 받는다.
2. 서치 파라미터로 URL을 업데이트한다.
3. URL을 입력 필드와 동기화 상태로 유지한다.
4. 서치 쿼리를 반영해 테이블을 업데이트한다.

유저의 입력을 URL에 반영시키는 방법을 알아보자.

```tsx
'use client';
...
export default function Search() {
	const searchParams = useSearchParams();
	const pathname = usePathname();
	const { replace } = useRouter();

	function handleSearch(term: string) {
		const params = new URLSearchParams(searchParams);
		if (term) {
	      params.set('query', term);
	  } else {
	    params.delete('query');
	  }
		replace(`${pathname}?${params.toString()}`);
	}
...
			<input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
				defaultValue={searchParams.get('query')?.toString()}
      />
...
```

input의 onChange를 통해 유저의 입력이 들어오면 handleSearch 함수로 입력을 전달한다.

handleSearch함수에서는 `useSearchParams`로 가져온 서치 파라미터들을 통해 새로운 URLSearchParams 객체를 만들어 유저의 입력을 넣어준다.

이후 `usePathname` 을 통해 가져온 pathname과 서치 파라미터들을 합쳐 URL을 만들고, `useRouter` 의 replace 함수를 통해 해당 URL로 유저를 이동시킨다.

input의 defaultValue 값을 서치 파라미터에서 가져온 값으로 지정해줌으로써 동기화시켜준다.

이렇게 유저의 입력을 서치 파라미터로 전달하고, URL을 변경시키는 방법을 알아보았다. 이제 URL에서 서치 파라미터들을 읽어와 사용하는 법을 알아보자.

---

```tsx
...
export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string;
    page?: string;
  };
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
...
			<Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
...
```

Page 컴포넌트들은 `searchParams` 라는 prop을 기본적으로 받을 수 있다. 이 prop을 통해 현재 URL의 서치 파라미터들에 접근하여 사용할 수 있다.

> **When to use the `useSearchParams()` hook vs the `searchParams` prop?**

서치 파라미터들을 추출하는 두가지 방법이 있다는 것을 확인했다. 둘 중에 어떤 것을 사용해야 할지는 현재 서치 파라미터를 활용하는 위치가 클라이언트 단인지, 서버 단인지에 달려있다.

- 앞서 `<Search>` 컴포넌트는 클라이언트 컴포넌트였고, 클라이언트에서 서치 파라미터에 접근하기 위해서는 `useSearchParams()` 훅을 사용해야한다.
- `<Table>` 컴포넌트는 데이터를 페치하는 서버 컴포넌트이고, page로부터 `searchParams` prop을 전달하여 사용할 수 있다.

일반적으로 클라이언트에서 서치 파라미터들을 읽고 싶다면 서버로 다시 돌아가는 현상을 방지하기 위해 `useSearchParams()` 훅을 사용한다.

>

## Best practice: Debouncing

현재는 유저의 매 키보드 입력 하나하나 마다 URL을 업데이트하고 있고, 이말인 즉 매 키보드 입력마다 DB에 쿼리를 보내고 있다는 말이다. 앱이 작고 유저가 적을때는 상관없지만, 유저수가 수천명만 넘어가도 매 키보드 입력 하나하나마다 DB에 쿼리를 보내는 것은 서버에 많은 부하를 줄 것이다.

이를 방지하기 위한 기법이 Debouncing으로, 함수가 실행되는 횟수를 제한하는 기법이다.

**Debouncing의 작동 원리**

1. 이벤트 발생 : 디바운스 되어야 할 이벤트(서치 박스의 키 입력과 같은)가 발생하면 타이머가 시작된다.
2. 대기 : 타이머가 종료되기 전에 새로운 이벤트가 발생한다면 타이머가 리셋된다.
3. 실행 : 타이머가 종료되면, 함수가 실행된다.

Debouncing을 구현하는 방법은 정말 다양하지만, 여기서는 간단히 하기 위해 `use-debounce` 라는 라이브러리를 사용한다.

```tsx
// ...
import { useDebouncedCallback } from 'use-debounce';

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```

이렇게 함수를 `useDebounceCallback` 이라는 훅으로 감싸주면 우리가 지정한 300ms의 타이머가 종료될 때, 즉 유저가 300ms동안 입력을 멈추고 기다릴 때만 해당 함수가 실행될 것이다.

# Chapter 12

## What are Server Actions?

React Server Action은 비동기 코드를 서버에서 직접 실행할 수 있게 해준다. 데이터를 변경해야할 때 API 엔드포인트를 만드는 대신 서버에서 동작하는 비동기 함수를 작성할 수 있게 해주는 것이다.

웹 어플리케이션은 다양한 위협에 취약하기 때문에 보안은 최우선 순위이다. Server Action은 여러 공격을 방어하고, 데이터를 안전하게하고, 허가된 접근을 보장하기 위해 효율적인 보안 솔루션을 제공한다. Server Action은 POST 요청, 암호화된 클로져들, 엄격한 입력 검사, 에러 메세지 해쉬, 호스트 제한 등을 이용해 우리의 앱이 안전하게 해준다.

## Next.js with Server Actions

Server Action은 Next.js의 캐싱과도 깊게 연관되어있다. form이 Server Action을 통해 제출되었을 때, data를 변경하는 뿐만아니라, `revalidatePath` 와 `revalidateTag` 와 같은 API들을 이용하는 것으로 연관된 캐시들을 다시 업데이트 할 수 있다.

## Create a Server Action

```tsx
'use server';
```

위 라인을 파일의 최상단에 작성하는 것으로, 해당 파일에서 export되는 모든 함수가 서버 함수임을 표시하는 것이다. 이 서버 함수들은 클라이언트 또는 서버 컴포넌트에 import되어 다양하게 사용될 수 있다.

물론 이렇게 파일을 따로 분리하는 것 뿐만 아니라 서버 컴포넌트의 action 내부에 “use server”를 추가하는 것으로 직접 Server Action을 작성할 수도 있다.

```tsx
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';

    // Logic to mutate data...
  }

  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

예를 들면 위와 같다.

## Validate and prepare the data

DB로 데이터를 전송하기 전에, 타입 검사를 확실하게 해줄 필요가 있다. 타입 검사를 구현하기 위해 여러 방법이 있지만, Zod 라이브러리를 사용하면 쉽고 효율적으로 코드를 작성할 수 있다.

```tsx
'use server';

import { z } from 'zod';

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

export async function createInvoice(formData: FormData) {
  // ...
}
```

이렇게 Zod 라이브러리를 통해 입력받을 form의 schema를 미리 정의해두고,

```tsx
// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
}
```

이런식으로 사용하면 된다. 타입 검사의 오류처리는 뒤에서 더 자세하게 알아보자. 추가적으로 Zod의 자세한 사용법은 공식 문서가 잘 설명하고 있으므로 해당 문서를 참고하자.

## Revalidate and redirect

form이 제출되면, 화면의 데이터를 변경하고 유저의 위치를 리다이렉트 시켜줄 필요가 있다. 화면의 데이터를 재평가하기 위해서 `revalidatePath` 를, 리다이렉트를 위해서 `redirect` 함수를 이용해주자.

```tsx
'use server';

import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

// ...

export async function createInvoice(formData: FormData) {
  // ...

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

## Create a Dynamic Route Segment with the invoice `id`

Next.js는 정확한 세그먼트 이름을 모르고, 데이터에 기반한 라우트를 만들고 싶을 때 다이나믹 라우트 세그먼트를 만들 수 있게해준다. 이는 블로그 포스트 제목, 제품 페이지 등등 일 수 있다. 폴더명을 대괄호로 감싸는 것으로 다이나믹 라우트 세그먼트를 만들 수 있다.

```tsx
// path : /app/dashboard/invoices/[id]/edit/page.tsx
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';

export default async function Page({ params }: { params: { id: string } }) {
  const id = params.id;
  // ...
}
```

마찬가지로 Page 컴포넌트의 `params` prop을 통해 파라미터를 읽어올 수 있다.

## Pass the `id` to the Server Action

`id` 값을 다음과 같이 인자로 전달할 수는 없다.

```tsx
// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

이를 가능케 하려면, JS의 `bind` 를 이용하면 된다.

```tsx
// ...
import { updateInvoice } from '@/app/lib/actions';

export default function EditInvoiceForm({
  invoice,
  customers,
}: {
  invoice: InvoiceForm;
  customers: CustomerField[];
}) {
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

  return (
    <form action={updateInvoiceWithId}>
      <input type="hidden" name="id" value={invoice.id} />
    </form>
  );
}
```

bind를 통해 인자를 전달한 새로운 함수를 리턴받아 사용하면 원하는 대로 `id` 를 전달할 수 있다.

# Chapter 13

## Adding `try/catch` to Server Actions

먼저 JS의 `try/catch` 구문을 이용해 Server Action에서 에러를 처리해보자.

```tsx
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  try {
    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    return {
      message: 'Database Error: Failed to Create Invoice.',
    };
  }

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

`redirect` 가 `try/catch` 블록의 밖에서 호출됨에 주목해보자. 이는 `redirect` 가 내부적으로 에러를 throw 함으로써 동작하기 때문이다. 만약 `try/catch` 블록 안에 `redirect` 가 존재한다면 `catch` 블록에 잡힐 것이다. 이를 방지하기 위해 `try/catch` 블록 밖에 선언해주고, `redirect` 는 `try` 가 성공적일 때만 호출될 것이다.

그렇다면 만약 에러가 발생한다면 어떻게 되는 것일까?

여기서 Next.js의 `error.tsx` 파일이 등장한다.

## Handling all errors with `error.tsx`

`error.tsx` 파일은 해당 라우트에서 발생할 수 있는 모든 에러를 잡아 fallback UI를 사용자에게 제공한다.

```tsx
'use client';

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Optionally log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>
      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
        onClick={
          // Attempt to recover by trying to re-render the invoices route
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```

몇가지 특징들을 살펴보자.

- **“use client”** - `error.tsx` 파일은 클라이언트 컴포넌트여야만 한다.
- 두개의 props를 전달받는다.
  - `error` - JS의 Error 객체의 인스턴스이다.
  - `reset` - 에러 바운더리를 리셋하는 함수이다. 실행되면 함수는 라우트 재렌더링을 시도할 것이다.

## Handling 404 errors with the `notFound` function

`error.tsx` 함수는 모든 에러를 잡아내지만, `notFound` 는 존재하지 않는 리소스를 요청했을 때 사용될 수 있다.

따로 처리해두지 않으면 존재하지 않는 리소스 요청에도 사용자는 `error.tsx` 에 정의된 fallback UI를 만나게 되지만, `notFound` 를 통해 좀 더 구체적인 정보를 알도록 해줄 수 있다.

```tsx
import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';
import { updateInvoice } from '@/app/lib/actions';
import { notFound } from 'next/navigation';

export default async function Page({ params }: { params: { id: string } }) {
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);

  if (!invoice) {
    notFound();
  }

  // ...
}
```

먼저 `notFound` 함수를 통해 존재하지 않는 리소스를 요청했을 때 404 에러를 발생하게 해준다.

![14](https://github.com/MinboyKim/nextjs-dashboard/assets/35567292/5a9d98b6-c2c2-4b41-b9ff-f311b960573c)

그 후 라우트 폴더 내에 `not-found.tsx` 파일을 통해 404 에러 발생 시 유저에게 보여줄 fallback UI를 정의할 수 있다.

# Chapter 14

## Form validation

### Client-Side validation

클라이언트 단에서 form의 유효성을 검증하는 방법은 여러가지가 있다. 가장 간단한 방법은 `<input>` 과 `<select>` 태그에 `required` 속성을 추가하는 것이다.

```tsx
<input
  id="amount"
  name="amount"
  type="number"
  placeholder="Enter USD amount"
  className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
  required
/>
```

이렇게하면 빈 input을 제출하려고 할 때 브라우저에서 오류를 보여준다.

### Server-Side validation

서버에서 form을 검증함으로써 다음과 같은 효과를 얻을 수 있다.

- 데이터를 DB로 보내기전에 형식을 확실하게 보장할 수 있다.
- 클라이언트 단의 검증을 우회하는 악의적인 사용자들의 위험을 줄일 수 있다.
- 유효한 데이터에 대한 하나의 정확성을 가질 수 있다.

```tsx
'use client';

// ...
import { useFormState } from 'react-dom';
```

`useFormState` 는 훅이기 때문에 `'use client'` 지시문으로 클라이언트 컴포넌트임을 명시해주어야한다.

Form 컴포넌트 내부에서, `useFormState` 훅은

- 두 개의 인자를 받는다. : `(action, initialState)`
- 두개의 값을 리턴한다. : `[state, dispatc]` - form state 와 dispatch 함수이다. (React의 useReducer 훅과 유사하다.)

기존의 action을 `useFormState` 의 인자로 넘겨주고, form의 action 속성에는 `dispatch` 함수를 전달해주자.

```tsx
// ...
import { useFormState } from 'react-dom';

export default function Form({ customers }: { customers: CustomerField[] }) {
  const initialState = { message: null, errors: {} };
  const [state, dispatch] = useFormState(createInvoice, initialState);

  return <form action={dispatch}>...</form>;
}
```

`initialState` 는 어떤 것도 될 수 있다.

Server Action쪽 코드에 작성되어 있던 `FormSchema` 를 수정해보자.

```tsx
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: 'Please select a customer.',
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: 'Please enter an amount greater than $0.' }),
  status: z.enum(['pending', 'paid'], {
    invalid_type_error: 'Please select an invoice status.',
  }),
  date: z.string(),
});
```

이제 Zod는 각 타입과 맞는지 검사해 맞지 않는다면 우리가 설정한 메세지를 리턴해줄 것이다. 이 메세지를 활용해 사용자에게 잘못되었음을 알려주자.

```tsx
// This is temporary until @types/react-dom is updated
export type State = {
  errors?: {
    customerId?: string[];
    amount?: string[];
    status?: string[];
  };
  message?: string | null;
};

export async function createInvoice(prevState: State, formData: FormData) {
  // ...
}
```

먼저 createInvoice 함수도 수정을 해주어야한다.

- `formData` - 이전과 동일
- `prevState` - `useFormState` 훅으로부터 전달된 state이다.

Zod의 `parse()` 부분도 `safeParse()` 로 변경해주어야한다.

```tsx
export async function createInvoice(prevState: State, formData: FormData) {
  // Validate form fields using Zod
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  // If form validation fails, return errors early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Missing Fields. Failed to Create Invoice.',
    };
  }

  // ...
}
```

`parse()`는 검증이 실패했을 때 오류를 throw 하지만, `safeParse()`는 성공 또는 실패를 포함한 객체를 리턴해준다. 이를 활용하여 오류 발생 시 유저에게 UI를 통해 잘못되었음을 알려줄 수 있다.

이제 다시 form으로 돌아와서 `state` 를 통해 오류를 감지하고 사용자에게 알려주자.

```tsx
<form action={dispatch}>
  <div className="rounded-md bg-gray-50 p-4 md:p-6">
    {/* Customer Name */}
    <div className="mb-4">
      <label htmlFor="customer" className="mb-2 block text-sm font-medium">
        Choose customer
      </label>
      <div className="relative">
        <select
          id="customer"
          name="customerId"
          className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
          defaultValue=""
          aria-describedby="customer-error"
        >
          <option value="" disabled>
            Select a customer
          </option>
          {customerNames.map((name) => (
            <option key={name.id} value={name.id}>
              {name.name}
            </option>
          ))}
        </select>
        <UserCircleIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500" />
      </div>
      <div id="customer-error" aria-live="polite" aria-atomic="true">
        {state.errors?.customerId &&
          state.errors.customerId.map((error: string) => (
            <p className="mt-2 text-sm text-red-500" key={error}>
              {error}
            </p>
          ))}
      </div>
    </div>
    // ...
  </div>
</form>
```

state와 삼항 연산자를 활용해 사용자에게 오류를 보여주는 것을 확인할 수 있다.

# Chapter 15

## What is authentication?

### Authentication vs Authorization

- Authentication(인증)은 유저가 자신을 누구라고 밝혔을 때 그 사실을 확인하는 것이다. 유저는 유저네임이나 패스워드등을 통해 자신의 신원을 제공한다.
- Authorization(허가)는 그 다음단계로, 유저의 신원이 확인되었을 때, 해당 유저가 앱의 어떤 부분을 사용할 수 있는지 결정하는 것이다.

즉 인증은 누구인지 신원을 체크하는 것, 허가는 해당 앱에서 어떤일을 할 수 있는지 결정하는 것을 의미한다.

## NextAuth.js

NextAuth.js는 세션 관리, 로그인 로그아웃 등 인증의 여러 복잡한 면들을 추상화하여 간단하고 쉽게 구현할 수 있게 해주는 라이브러리이다.

```bash
npm install next-auth@beta
```

Next.js 14 버전과 함께 사용하기 위해서는 `beta` 버전을 설치해주어야한다.

```bash
openssl rand -base64 32
```

터미널에서 opennssl을 통해 암호키를 생성해주고 .env 파일에 `AUTH_SECRET=my-secret-key` 로 추가해주자. 해당 암호키를 통해 쿠키를 암호화하고, 유저 세션의 보안에 이용할 것이다.

또한 배포에서 인증을 작동시키기 위해 Vercel에서도 환경변수를 업데이트 해주어야한다. 업데이트하는 법은 Vercel의 공식문서를 참조하자.

## Adding the pages option

```tsx
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
};
```

프로젝트의 최상단에 `auth.config.ts` 파일을 생성해주자. 여기서 export 하는 `authConfig` 객체는 NextAuth.js를 위한 설정 옵션들을 포함하고 있다. 현재는 `pages` 옵션만 갖고있다.

`pages` 옵션을 통해 커스텀 로그인, 로그아웃, 에러 페이지들을 위한 라우트를 명시할 수 있다. 이것은 필수 옵션은 아니지만 `pages` 옵션에 `signIn: '/login'` 을 추가해주는것으로 유저는 NextAuth.js의 디폴트 페이지가 아닌 우리의 커스텀 로그인 페이지로 리다이렉트될 것이다.

## Protecting your routes with Next.js Middleware

라우트를 보호하기 위한 로직을 추가해보자. 이를 통해 대쉬보드 페이지에 로그인하지 않은 사용자가 접근하는 것을 방지하도록 만들어보자.

```tsx
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }
      return true;
    },
  },
  providers: [], // Add providers with an empty array for now
} satisfies NextAuthConfig;
```

`authorized` 콜백은 요청이 Next.js 미들웨어를 통한 접근이 허가되었는지 확인하기 위해 사용되었다. 해당 콜백은 요청이 완료되기 전에 호출되고, `auth` 와 `request` 프로퍼티가 존재하는 객체를 전달받는다. `auth` 프로퍼티는 유저의 세션을 포함하고 있고, `request` 프로퍼티는 들어오는 요청이 포함되어있다.

`provider` 옵션은 다른 로그인 옵션들을 리스트하는 배열이다. 현재는 비어있지만 나중에 더 살펴보자.

다음으로, 이제 이 `authConfig` 객체를 미들웨어 파일에서 import 해보자. 프로젝트의 루트에서 `middleware.ts` 파일을 생성하고 다음 코드를 붙혀넣자.

```tsx
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
  // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

여기서 `authConfig` 객체와 함께 NextAuth.js를 초기화하고, 동시에 `auth` 프로퍼티를 export하고 있다. 미들웨어의 `matcher` 옵션을 통해 미들웨어가 수행될 path를 명시할 수 있다.

## Password hashing

DB에 패스워드를 저장하기 전에 해싱하는 것은 좋은 관습이다. 해싱은 패스워드를 랜덤해보이는 고정길이의 문자열로 변환해주며, 유저의 데이터가 노출되어도 한층 더 안전하게 해준다.

이전에 사용했던 `bcrypt` 패키지를 다시 사용하여 해싱을 처리할 것이다. 하지만 `bcrypt` 패키지는 Next.js 미들웨어에서 사용 불가능한 Node.js API에 의존하기 때문에 파일을 분리해주어야한다.

```tsx
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

`authConfig` 객체를 spread하는 `auth.ts` 파일을 생성해주자.

## Adding the Credentials provider

`provider` 옵션은 상기한대로 구글, 깃허브와 같은 다른 로그인 옵션들을 리스트해두는 배열이다. 여기서는 `Credentials provider` 만 사용할 것이다.

`Credentials provider` 는 유저가 유저네임과 패스워드를 통해 로그인할 수 있도록 해준다.

```tsx
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [Credentials({})],
});
```

> 알면 좋은 사실 : 여기서는 Credentials provider만 사용하지만, 일반적으로는 OAuth나 email provider들을 통한 로그인 대안 방법들을 함께 사용하는 것을 추천한다.

## Adding the sign in functionality

`authorize` 함수를 통해 인증 로직을 핸들링 할 수 있다. Server Actions 처럼, `zod` 를 이용해 이메일과 패스워드가 DB에 존재하는지 확인하기 전에 먼저 검증할 수 있다.

```tsx
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
      },
    }),
  ],
});
```

검증 후, DB에 해당 유저가 존재하는지 쿼리를 보내는 `getUser` 함수를 만들어주자.

```tsx
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User>`SELECT * FROM users WHERE email=${email}`;
    return user.rows[0];
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw new Error('Failed to fetch user.');
  }
}

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);

        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
        }

        return null;
      },
    }),
  ],
});
```

이제 `bcrypt.compare` 를 사용해 비밀번호가 일치하는지 확인하자.

```tsx
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { sql } from '@vercel/postgres';
import { z } from 'zod';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

// ...

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        // ...

        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
          const passwordsMatch = await bcrypt.compare(password, user.password);

          if (passwordsMatch) return user;
        }

        console.log('Invalid credentials');
        return null;
      },
    }),
  ],
});
```

마침내, 만약 비밀번호가 일치한다면 유저를 리턴받을 것이고, 그렇지 않으면 유저가 로그인하는 것을 방지하기 위해 `null` 을 리턴받을 것이다.

## Updating the login form

이제 인증 로직을 로그인 form과 연결해주어야한다. `action.ts` 파일에서, `authenticate` 라는 새로운 액션을 만들어주자. 이 액션은 `auth.ts` 에서 `signIn` 함수를 import 해와야만 한다.

```tsx
import { signIn } from '@/auth';
import { AuthError } from 'next-auth';

// ...

export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```

마지막으로, `login-form.tsx` 컴포넌트에서, React의 `useFormState` 를 사용해 Server Action을 호출하고 form 에러를 핸들링하고, `useFormStatus` 를 통해 form의 pending state를 핸들링하자.

```tsx
'use client';

import { lusitana } from '@/app/ui/fonts';
import {
  AtSymbolIcon,
  KeyIcon,
  ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import { ArrowRightIcon } from '@heroicons/react/20/solid';
import { Button } from '@/app/ui/button';
import { useFormState, useFormStatus } from 'react-dom';
import { authenticate } from '@/app/lib/actions';

export default function LoginForm() {
  const [errorMessage, dispatch] = useFormState(authenticate, undefined);

  return (
    <form action={dispatch} className="space-y-3">
				//...
				<div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )}
        </div>
      </div>
    </form>
  );
}

function LoginButton() {
  const { pending } = useFormStatus();

  return (
    <Button className="mt-4 w-full" aria-disabled={pending}>
      Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
    </Button>
  );
}
```

## Adding the logout functionality

로그아웃 로직을 `<SideNav />` 에 추가하기 위해, `signOut` 함수를 `auth.ts` 파일에서 import 해와 `<form>` 의 action에서 호출해주자.

```tsx
import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import { PowerIcon } from '@heroicons/react/24/outline';
import { signOut } from '@/auth';

export default function SideNav() {
  return (
    <div className="flex h-full flex-col px-3 py-4 md:px-2">
      // ...
      <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
        <NavLinks />
        <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
        <form
          action={async () => {
            'use server';
            await signOut();
          }}
        >
          <button className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3">
            <PowerIcon className="w-6" />
            <div className="hidden md:block">Sign Out</div>
          </button>
        </form>
      </div>
    </div>
  );
}
```

# Chapter 16

## What is metadata?

메타데이터란 페이지에 방문하는 유저들에게는 보이지 않지만, 페이지의 HTML 속(주로 `<head>` 엘리먼트 속)에 포함되어있는 웹페이지의 추가적인 정보들을 말한다. 이 숨겨진 정보들은 서치엔진과 같은 시스템들에게 우리의 웹페이지 내용들을 더 잘 이해시키기 위해 매우 중요하다.

## Why is metadata important?

메타데이터는 웹페이지의 SEO에서 중대한 역할을 차지하고 있다. 서치 엔진들과 소셜 미디어 플랫폼들이 페이지에 더 잘 접근하고 더 잘 이해할 수 있게 해주는 역할을 수행한다. 적절한 메타데이터는 서치엔진이 효율적으로 웹페이지들의 색인을 생성할 수 있게해줌으로써 검색 결과의 랭킹을 향상시키는데 도움을 준다. 추가적으로 Open Graph와 같은 메타데이터는 소셜 미디어에서 공유 링크의 모습을 향상시켜줌으로써 유저들에게 좀 더 내용을 잘 전달시킬 수 있다.

## Adding metadata

Next.js는 앱의 메타데이터를 정의하는데 사용되는 Meatadata API들을 갖고있다. 앱에 메타데이터를 추가하는데는 두가지 방법이 존재한다.

- Config-based : `layout.js` 또는 `page.js` 파일에서 정적 메타데이터 객체 또는 동적 generateMetadata 함수를 export 해주기
- File-based : Next.js는 메타데이터를 명시하기 위한 일련의 파일들을 갖고 있다.
  - `favicon.ico`, `apple-icon.jpg` , `icon.jpg` : 파비콘 및 아이콘 활성화
  - `opengraph-image.jpg` , `twitter-image.jpg` : 소셜 미디어 이미지 적용
  - `robots.txt` : 서치 엔진 크롤링을 위한 지시 제공
  - `sitemap.xml` : 웹사이트 구조에 대한 정보 제공

이 모든 방법들을 유연하게 사용할 수 있다. 이 옵션들을 통해 Next.js는 자동적으로 적절한 `<head>` 엘리먼트를 생성해줄 것이다.

## Page title and descriptions

예시로 `layout.js` 에서 title 메타데이터를 정의하는 방법을 알아보자. 해당 `layout.js` 에 존재하는 모든 메타데이터들은 그 레이아웃을 사용하는 모든 페이지에 상속될 것이다.

루트 레이아웃에서 다음 코드를 작성하자.

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Acme Dashboard',
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};

export default function RootLayout() {
  // ...
}
```

Next.js가 자동적으로 메타데이터와 title을 추가해줄 것이다.

만약 특정 페이지에 커스텀 타이틀을 추가하고 싶다면 어떻게 해야할까? 그냥 해당 페이지의 `page.tsx` 에서 `metatdata` 객체를 추가해주면된다. 예를들어 `/dashboard/invoices` 페이지의 페이지 타이틀을 업데이트 하고 싶다면

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Invoices | Acme Dashboard',
};
```

이 코드를 추가해주면된다.

이 방식은 잘 작동하지만, 모든 페이지마다 타이틀을 반복하는 것은 만약 회사명이 바뀌었다던가 뭔가가 변화했을 때 하나하나 일일히 변경해주어야하는 번거로움이 존재한다.

대신에 `title.template` 필드를 `metadata` 객체에 정의해주는 것으로 페이지 타이틀에 대한 템플릿을 정의할 수 있다.

루트 레이아웃에서 다음과 같이 작성해보자.

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

이제 하위 페이지들에서는 `%s` 템플릿을 지정해주기만 하면된다.

예를들어 `/dashboard/invoices` 페이지에서는 다음과 같이 사용하면 된다.

```tsx
export const metadata: Metadata = {
  title: 'Invoices',
};
```

이렇게만 하면 `Invoices | Acme Dasboard` 라는 타이틀을 확인할 수 있다.
