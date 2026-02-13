---
title: Frontend Architecture
subject: "[[Dev Note]]"
reference: ""
date: 2026-02-13 11:22
description: 2026년 프론트엔드 아키텍처 트렌드와 실용적인 패턴 가이드
tags:
  - architecture
  - frontend
  - clean-architecture
  - islands
  - ai-native-architecture
  - vertical-slice-architecture
  - fsd
  - ai
  - 개념
---

# 프론트엔드 아키텍처 (2026 Trends)

2026년 현재 프론트엔드 생태계는 단순히 화면을 그리는 것을 넘어, 대규모 애플리케이션의 유지보수성과 AI와의 협업 효율성을 극대화하는 방향으로 진화했습니다.

## 1. Feature-Sliced Design (FSD)

러시아의 대규모 커머스 팀들로부터 시작되어 전 세계적으로 표준이 된 아키텍처입니다. **비즈니스 가치**를 중심으로 코드를 분리하는 것이 핵심입니다.

### 구조 (Layers)
레이어는 위에서 아래로만 의존성을 가질 수 있습니다. (예: `Pages`는 `Widgets`을 import 가능, 반대는 불가)

1.  **Shared**: 특정 비즈니스 로직에 종속되지 않는 재사용 가능한 코드 (UI Kit, libs, API client).
2.  **Entities**: 비즈니스 도메인 (User, Product, Order).
3.  **Features**: 사용자 상호작용 (AddToCart, Login, LikePost).
4.  **Widgets**: Feature와 Entity를 결합한 독립적인 UI 블록 (Header, ProductList).
5.  **Pages**: 위젯들을 조합하여 구성된 실제 화면.
6.  **Processes**: (선택적) 여러 페이지에 걸친 복잡한 프로세스 (Checkout, Registration).
7.  **App**: 전역 설정, Provider, Entry point.

### 실제 구현 예시
```typescript
// ❌ 잘못된 의존성 (상위 레이어가 하위 레이어를 import)
// features/add-to-cart/ui/AddToCartButton.tsx
import { ProductPage } from '@/pages/product' // Pages는 Features보다 상위!

// ✅ 올바른 의존성
// pages/product/ui/ProductPage.tsx
import { AddToCartButton } from '@/features/add-to-cart'
import { ProductCard } from '@/entities/product'
import { Header } from '@/widgets/header'

export const ProductPage = () => {
  return (
    <>
      <Header />
      <ProductCard>
        <AddToCartButton />
      </ProductCard>
    </>
  )
}
```

### FSD 폴더 구조 예시
```
src/
  app/
    providers/          # 전역 Provider (Auth, Theme, etc)
    styles/            # 전역 스타일
    config/            # 환경 변수, 라우팅 설정
  pages/
    product/
      ui/ProductPage.tsx
      model/useProductPage.ts
    checkout/
  widgets/
    header/
      ui/Header.tsx
      model/useHeader.ts
    product-list/
  features/
    add-to-cart/
      ui/AddToCartButton.tsx
      api/addToCartMutation.ts
      model/useAddToCart.ts
    auth-by-email/
  entities/
    product/
      ui/ProductCard.tsx
      model/product.types.ts
      api/productApi.ts
    user/
  shared/
    ui/              # Button, Input 등 UI Kit
    lib/             # 유틸 함수
    api/             # API 클라이언트 설정
    config/          # 상수
```

### 장점
-   **확장성**: 기능이 많아져도 복잡도가 선형적으로만 증가합니다.
-   **명확한 경계**: 팀 간의 충돌을 줄이고 모듈화가 강력합니다.

---

## 2. Server Components & Islands Architecture

Next.js와 React Server Components(RSC)가 정착되면서 프론트엔드 아키텍처의 기준이 되었습니다.

### 개념
-   **기본은 서버**: 가능한 모든 컴포넌트를 서버에서 렌더링하여 JS 번들 크기를 0으로 만듭니다.
-   **Islands (상호작용 섬)**: `use client` 지시어를 사용해 사용자와 상호작용이 꼭 필요한 부분(버튼, 폼 등)만 클라이언트 컴포넌트로 만듭니다.
-   **Streaming**: 전체 페이지를 기다리지 않고 준비된 부분부터 사용자에게 전송합니다.

### 장점
-   **성능**: 초기 로딩 속도(LCP)와 검색 엔진 최적화(SEO)가 압도적입니다.
-   **보안**: 민감한 키나 로직을 서버 컴포넌트에 숨길 수 있습니다. (BFF 패턴의 내재화)

### 실제 구현 예시
```typescript
// app/product/[id]/page.tsx (Server Component - 기본)
import { db } from '@/lib/db'
import { AddToCartButton } from './AddToCartButton' // Client Component

export default async function ProductPage({ params }: { params: { id: string } }) {
  // 서버에서 직접 DB 접근 - 클라이언트로 전송되지 않음
  const product = await db.product.findUnique({
    where: { id: params.id },
    include: { reviews: true }
  })

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
      {/* 상호작용이 필요한 부분만 Client Component */}
      <AddToCartButton productId={product.id} />
    </div>
  )
}

// AddToCartButton.tsx (Client Component)
'use client' // 이 지시어가 있어야 useState, onClick 등 사용 가능

import { useState } from 'react'

export function AddToCartButton({ productId }: { productId: string }) {
  const [isLoading, setIsLoading] = useState(false)

  const handleClick = async () => {
    setIsLoading(true)
    await fetch('/api/cart', { method: 'POST', body: JSON.stringify({ productId }) })
    setIsLoading(false)
  }

  return <button onClick={handleClick}>{isLoading ? '추가 중...' : '장바구니 담기'}</button>
}
```

### 성능 최적화: Streaming + Suspense
```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react'
import { UserProfile } from './UserProfile'
import { RecentOrders } from './RecentOrders'
import { Recommendations } from './Recommendations'

export default function DashboardPage() {
  return (
    <div>
      {/* 빠른 컴포넌트는 즉시 렌더링 */}
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile />
      </Suspense>

      {/* 느린 API는 독립적으로 Streaming */}
      <Suspense fallback={<OrdersSkeleton />}>
        <RecentOrders />
      </Suspense>

      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations /> {/* ML 추천 API - 5초 걸려도 나머지는 먼저 보임 */}
      </Suspense>
    </div>
  )
}
```

---

## 3. Clean Architecture (Hexagonal Architecture)

프레임워크(React, Vue 등)와 비즈니스 로직을 철저히 분리하고 싶을 때 사용합니다.

### 구조
-   **Domain Layer**: 엔티티와 비즈니스 규칙. 프레임워크를 전혀 모르는 순수 JS/TS 코드.
-   **Use Case Layer**: 애플리케이션의 특정 동작 흐름.
-   **Adapter Layer**: UI(React 컴포넌트)나 외부 서비스(API)와 도메인 사이를 연결.
-   **Framework Layer**: React, Vue, Browser API 등.

### 실제 구현 예시
```typescript
// ❌ 도메인 로직이 React에 강하게 결합된 코드
function CheckoutForm() {
  const [cart, setCart] = useState([])

  const calculateTotal = () => {
    let total = 0
    cart.forEach(item => {
      total += item.price * item.quantity
      if (item.category === 'electronics') {
        total *= 0.9 // 전자제품 10% 할인
      }
    })
    if (total > 100000) total -= 5000 // 10만원 이상 5천원 할인
    return total
  }
  // ... UI 코드와 비즈니스 로직이 뒤섞임
}

// ✅ Clean Architecture로 분리
// domain/order/OrderCalculator.ts (순수 TypeScript - React 몰라도 됨)
export class OrderCalculator {
  calculateTotal(items: OrderItem[]): number {
    const subtotal = this.calculateSubtotal(items)
    const discountedTotal = this.applyItemDiscounts(subtotal, items)
    return this.applyOrderDiscounts(discountedTotal)
  }

  private calculateSubtotal(items: OrderItem[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  }

  private applyItemDiscounts(total: number, items: OrderItem[]): number {
    const electronicsCount = items.filter(i => i.category === 'electronics').length
    return electronicsCount > 0 ? total * 0.9 : total
  }

  private applyOrderDiscounts(total: number): number {
    return total > 100000 ? total - 5000 : total
  }
}

// domain/order/OrderCalculator.spec.ts (테스트하기 쉬움!)
describe('OrderCalculator', () => {
  it('should apply electronics discount', () => {
    const calculator = new OrderCalculator()
    const items = [{ price: 10000, quantity: 2, category: 'electronics' }]
    expect(calculator.calculateTotal(items)).toBe(18000) // 20000 * 0.9
  })
})

// presentation/CheckoutForm.tsx (UI는 얇게)
import { OrderCalculator } from '@/domain/order/OrderCalculator'

function CheckoutForm() {
  const [cart] = useCart()
  const calculator = useMemo(() => new OrderCalculator(), [])
  const total = calculator.calculateTotal(cart)

  return <div>총액: {total}원</div>
}
```

### 2026년의 변화: AI와의 시너지
AI가 코드를 작성할 때 도메인 로직이 UI에 섞여 있으면 환각(Hallucination)이 발생하기 쉽습니다. 클린 아키텍처로 로직을 순수 함수로 분리해두면 AI가 테스트 코드를 작성하거나 로직을 수정하기 훨씬 수월해집니다.

**AI 활용 예시:**
```bash
# AI에게 도메인 로직만 전달하면 정확한 테스트 생성
$ claude "OrderCalculator.ts를 읽고 엣지 케이스 테스트 5개 추가해줘"

# UI 코드 없이 비즈니스 규칙만 수정 가능
$ claude "10만원 → 15만원으로 할인 기준 변경해줘"
```

---

## 4. AI-Native Architecture (Context-Optimized)

LLM(거대언어모델)이 코드를 더 잘 이해하고 수정할 수 있도록 최적화된 최신 아키텍처 트렌드입니다.

### 핵심 원칙
1.  **Co-location (위치 통합)**: 관련된 코드, 테스트, **문서(.md)**, 타입 정의를 한 폴더에 모아둡니다. AI가 Context Window를 효율적으로 쓰도록 돕습니다.
2.  **Explicit Typing**: 타입을 통해 AI에게 명확한 제약 조건을 줍니다. `any` 사용은 AI의 추론 능력을 떨어뜨리므로 엄격히 금지합니다.
3.  **Small Functions**: 함수는 한 가지 일만 하도록 극도로 작게 쪼갭니다. AI가 한 번에 완벽하게 이해하고 수정하기 좋습니다.
4.  **Self-Documenting Code**: 주석 대신 변수명과 함수명을 서술적으로(Descriptive) 작성합니다.

### 예시 구조
```
src/
  features/
    auth/
      login/
        LoginButton.tsx
        login.action.ts   # 서버 액션
        login.spec.ts     # 테스트
        README.md         # AI를 위한 로직 설명
```

### AI 최적화 실전 가이드

#### 1. Co-location 실전 예시
```typescript
// features/payment/process-payment/
// ├── README.md                 ← AI가 가장 먼저 읽을 문서
// ├── processPayment.ts         ← 핵심 로직
// ├── processPayment.spec.ts    ← 테스트
// ├── types.ts                  ← 타입 정의
// └── errors.ts                 ← 에러 정의

// README.md 예시
/*
# 결제 처리 로직

## 역할
PG사 API를 호출하여 결제를 처리합니다.

## 제약사항
- 금액은 100원 이상이어야 함
- 동시에 같은 주문 ID로 여러 번 호출 불가 (멱등성)

## 에러 처리
- `PaymentAmountError`: 금액이 100원 미만
- `DuplicatePaymentError`: 중복 결제 시도
- `PGApiError`: PG사 API 오류
*/

// processPayment.ts
export async function processPayment(
  order: Order,
  paymentMethod: PaymentMethod
): Promise<PaymentResult> {
  validatePaymentAmount(order.amount)
  await checkDuplicatePayment(order.id)
  return await callPGApi(order, paymentMethod)
}

// ✅ AI가 README를 읽고 테스트를 자동 생성 가능
// ✅ AI가 에러 처리를 완벽하게 이해
```

#### 2. Explicit Typing - AI 추론 최적화
```typescript
// ❌ AI가 혼란스러워하는 코드
function calculate(data: any, opts?: any) {
  return data.reduce((a: any, b: any) => a + b.val * (opts?.tax || 1), 0)
}

// ✅ AI가 완벽하게 이해하는 코드
type Product = {
  name: string
  price: number
  taxable: boolean
}

type CalculationOptions = {
  includeTax: boolean
  taxRate: number
}

function calculateTotalPrice(
  products: Product[],
  options: CalculationOptions
): number {
  return products.reduce((total, product) => {
    const basePrice = product.price
    const tax = product.taxable && options.includeTax
      ? basePrice * options.taxRate
      : 0
    return total + basePrice + tax
  }, 0)
}

// AI에게 요청: "taxRate를 product별로 다르게 적용하도록 수정해줘"
// → 타입 덕분에 AI가 정확히 수정 가능
```

#### 3. Small Functions - AI 컨텍스트 최적화
```typescript
// ❌ AI가 한 번에 이해하기 어려운 거대한 함수 (100줄)
function handleCheckout(cart, user, shipping, payment, coupon) {
  // 장바구니 검증
  // 재고 확인
  // 쿠폰 적용
  // 배송비 계산
  // 결제 처리
  // 주문 생성
  // 이메일 발송
  // ... 100줄
}

// ✅ AI가 각각 완벽하게 이해하고 수정할 수 있는 작은 함수들
async function validateCartItems(cart: Cart): Promise<ValidationResult> {
  // 5줄
}

async function checkInventory(items: CartItem[]): Promise<InventoryStatus> {
  // 7줄
}

function applyCoupon(subtotal: number, coupon: Coupon): number {
  // 3줄
}

function calculateShipping(address: Address, weight: number): number {
  // 8줄
}

// Orchestrator (조합 함수)
async function processCheckout(request: CheckoutRequest): Promise<Order> {
  const validation = await validateCartItems(request.cart)
  if (!validation.isValid) throw new ValidationError(validation.errors)

  const inventory = await checkInventory(request.cart.items)
  if (!inventory.available) throw new OutOfStockError(inventory.unavailableItems)

  const subtotal = calculateSubtotal(request.cart)
  const discount = applyCoupon(subtotal, request.coupon)
  const shipping = calculateShipping(request.address, request.cart.totalWeight)

  return await createOrder({ subtotal, discount, shipping })
}

// ✅ AI에게 "쿠폰 로직만 수정해줘" 요청 시 applyCoupon만 정확히 수정
```

#### 4. AI와 페어 프로그래밍 워크플로우
```bash
# 1단계: AI가 타입 정의 생성
$ claude "결제 시스템에 필요한 타입들을 types.ts에 정의해줘.
  Payment, PaymentMethod, PaymentStatus, PaymentResult 포함"

# 2단계: AI가 README.md 기반으로 구현
$ claude "README.md를 읽고 processPayment 함수를 구현해줘.
  모든 제약사항과 에러 처리를 포함해야 해"

# 3단계: AI가 테스트 생성
$ claude "processPayment.ts를 읽고 모든 엣지 케이스를
  커버하는 테스트를 작성해줘"

# 4단계: AI가 리팩토링
$ claude "processPayment.ts에서 10줄 이상인 함수를 찾아서
  5줄 이하의 작은 함수들로 분리해줘"
```

#### 5. Self-Documenting Code 예시
```typescript
// ❌ 주석에 의존 (AI가 주석과 코드 불일치 발견 못함)
function calc(u: User, o: Order) {
  // 유저가 VIP면 10% 할인
  const d = u.t === 'vip' ? 0.1 : 0
  return o.amt * (1 - d)
}

// ✅ 코드 자체가 문서 (AI가 정확히 이해)
function calculateOrderTotalWithMembershipDiscount(
  user: User,
  order: Order
): number {
  const discountRate = getDiscountRateByMembershipTier(user.membershipTier)
  const discountAmount = order.amount * discountRate
  return order.amount - discountAmount
}

function getDiscountRateByMembershipTier(tier: MembershipTier): number {
  const DISCOUNT_RATES = {
    VIP: 0.1,
    PREMIUM: 0.05,
    STANDARD: 0
  }
  return DISCOUNT_RATES[tier]
}
```

#### 6. AI-Friendly 에러 처리
```typescript
// ❌ AI가 이해하기 어려운 에러 처리
function processOrder(order) {
  try {
    // ... 복잡한 로직
  } catch (e) {
    console.error(e)
    return null // 뭔가 실패했는데 AI가 모름
  }
}

// ✅ AI가 완벽하게 이해하는 명시적 에러
class OrderProcessingError extends Error {
  constructor(
    message: string,
    public readonly code: OrderErrorCode,
    public readonly orderID: string,
    public readonly cause?: Error
  ) {
    super(message)
    this.name = 'OrderProcessingError'
  }
}

async function processOrder(order: Order): Promise<ProcessedOrder> {
  if (order.amount < MINIMUM_ORDER_AMOUNT) {
    throw new OrderProcessingError(
      `Order amount ${order.amount} is below minimum ${MINIMUM_ORDER_AMOUNT}`,
      'AMOUNT_TOO_LOW',
      order.id
    )
  }

  try {
    return await paymentGateway.process(order)
  } catch (error) {
    throw new OrderProcessingError(
      'Payment gateway failed',
      'PAYMENT_GATEWAY_ERROR',
      order.id,
      error as Error
    )
  }
}

// AI가 에러 처리 추가 요청 시 정확히 수행 가능
```

---

## 5. Vertical Slice Architecture

2026년 마이크로서비스 팀에서 급부상한 패턴입니다. **기능별로 수직으로 슬라이싱**하여 각 기능이 독립적인 전체 스택을 가집니다.

### 핵심 개념
- FSD와 유사하지만 더 극단적으로 **기능 단위 독립성** 강조
- 각 슬라이스는 자체 DB 쿼리, API, 상태 관리, UI를 모두 포함
- 수평적 레이어(Controller/Service/Repository)가 아닌 수직적 기능 단위

### 구조 예시
```
src/
  slices/
    user-registration/
      ui/
        RegistrationForm.tsx
      api/
        registerUser.ts
        checkEmailAvailability.ts
      state/
        registrationStore.ts
      validation/
        registrationSchema.ts
      db/
        userQueries.ts       # 이 기능만을 위한 쿼리
    product-search/
      ui/
        SearchBar.tsx
        SearchResults.tsx
      api/
        searchProducts.ts
      state/
        searchStore.ts
      algolia/               # 이 기능만의 검색 엔진 통합
        algoliaClient.ts
```

### 장점
- **팀 자율성**: 각 팀이 슬라이스 하나를 완전히 소유
- **배포 독립성**: 한 슬라이스 수정이 다른 슬라이스에 영향 없음
- **AI 협업 최적화**: AI에게 "user-registration 슬라이스만 수정해줘"라고 명확히 지시 가능

### 실제 코드 예시
```typescript
// slices/add-to-cart/add-to-cart.slice.ts
// 이 파일 하나가 전체 기능을 담당

import { create } from 'zustand'
import { cartApi } from './api/cartApi'

// 상태
type CartState = {
  items: CartItem[]
  isAdding: boolean
  error: string | null
}

// 액션
type CartActions = {
  addItem: (productId: string, quantity: number) => Promise<void>
  removeItem: (itemId: string) => Promise<void>
}

// 스토어
export const useCartStore = create<CartState & CartActions>((set) => ({
  items: [],
  isAdding: false,
  error: null,

  addItem: async (productId, quantity) => {
    set({ isAdding: true, error: null })
    try {
      const newItem = await cartApi.addItem(productId, quantity)
      set((state) => ({ items: [...state.items, newItem], isAdding: false }))
    } catch (error) {
      set({ error: error.message, isAdding: false })
    }
  },

  removeItem: async (itemId) => {
    await cartApi.removeItem(itemId)
    set((state) => ({
      items: state.items.filter((item) => item.id !== itemId)
    }))
  }
}))

// UI 컴포넌트
export function AddToCartButton({ productId }: { productId: string }) {
  const { addItem, isAdding } = useCartStore()

  return (
    <button onClick={() => addItem(productId, 1)} disabled={isAdding}>
      {isAdding ? '추가 중...' : '장바구니 담기'}
    </button>
  )
}
```

---

## 6. Micro-Frontend Architecture

대규모 조직에서 여러 팀이 독립적으로 프론트엔드를 개발할 때 사용합니다.

### 통합 방식

#### 1. Module Federation (Webpack 5 / Rspack)
```javascript
// host-app/rspack.config.js (메인 앱)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        productApp: 'productApp@http://localhost:3001/remoteEntry.js',
        cartApp: 'cartApp@http://localhost:3002/remoteEntry.js'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
}

// host-app/src/App.tsx
import { lazy, Suspense } from 'react'

const ProductList = lazy(() => import('productApp/ProductList'))
const CartWidget = lazy(() => import('cartApp/CartWidget'))

export default function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <ProductList />
        <CartWidget />
      </Suspense>
    </div>
  )
}
```

#### 2. Web Components (프레임워크 독립적)
```typescript
// cart-app/src/cart-widget.ts (Vue로 작성)
import { defineCustomElement } from 'vue'
import CartWidget from './CartWidget.vue'

const CartWidgetElement = defineCustomElement(CartWidget)
customElements.define('cart-widget', CartWidgetElement)

// host-app (React 앱에서 사용)
export default function App() {
  return (
    <div>
      <h1>My Shop</h1>
      <cart-widget user-id="123" />  {/* Vue 컴포넌트를 React에서 사용 */}
    </div>
  )
}
```

### 장점
- **기술 스택 자유도**: 팀마다 React, Vue, Svelte 등 선택 가능
- **독립 배포**: 각 마이크로 프론트엔드를 독립적으로 배포

### 주의사항
- **복잡도 증가**: 작은 프로젝트에서는 과도한 오버엔지니어링
- **번들 중복**: Shared 설정 없으면 React가 여러 번 로드될 수 있음

---

## 7. Domain-Driven Design (DDD) for Frontend

백엔드의 DDD를 프론트엔드에 적용한 패턴입니다.

### 핵심 개념
- **Bounded Context**: 각 도메인 영역을 명확히 분리 (주문, 배송, 결제)
- **Ubiquitous Language**: 도메인 전문가와 개발자가 같은 용어 사용
- **Aggregate**: 일관성을 유지해야 하는 엔티티들의 그룹

### 구조 예시
```
src/
  domains/
    order/                    # Bounded Context: 주문
      aggregates/
        Order.ts              # Aggregate Root
        OrderItem.ts
        OrderStatus.ts
      value-objects/
        Money.ts
        Address.ts
      repositories/
        OrderRepository.ts
      services/
        OrderService.ts
      ui/
        OrderList.tsx
        OrderDetail.tsx
    payment/                  # Bounded Context: 결제
      aggregates/
        Payment.ts
      value-objects/
        CardNumber.ts
      services/
        PaymentService.ts
```

### 실제 구현 예시
```typescript
// domains/order/aggregates/Order.ts
export class Order {
  private constructor(
    public readonly id: OrderID,
    public readonly items: OrderItem[],
    private status: OrderStatus,
    public readonly totalAmount: Money
  ) {}

  // 도메인 로직을 Aggregate 안에 캡슐화
  public cancel(): void {
    if (this.status === OrderStatus.Shipped) {
      throw new Error('이미 배송된 주문은 취소할 수 없습니다')
    }
    this.status = OrderStatus.Cancelled
  }

  public addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.Pending) {
      throw new Error('진행 중인 주문에만 상품을 추가할 수 있습니다')
    }
    const newItem = OrderItem.create(product, quantity)
    this.items.push(newItem)
  }

  // Factory Method
  public static create(items: OrderItem[]): Order {
    const totalAmount = items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero()
    )
    return new Order(
      OrderID.generate(),
      items,
      OrderStatus.Pending,
      totalAmount
    )
  }
}

// domains/order/value-objects/Money.ts
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: Currency
  ) {
    if (amount < 0) throw new Error('금액은 음수일 수 없습니다')
  }

  public add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('다른 통화는 더할 수 없습니다')
    }
    return new Money(this.amount + other.amount, this.currency)
  }

  public static zero(): Money {
    return new Money(0, Currency.KRW)
  }
}

// domains/order/ui/OrderDetail.tsx
import { useOrder } from '../hooks/useOrder'

export function OrderDetail({ orderId }: { orderId: string }) {
  const { order, cancelOrder } = useOrder(orderId)

  const handleCancel = async () => {
    try {
      order.cancel() // 도메인 로직 호출
      await orderRepository.save(order)
    } catch (error) {
      alert(error.message) // "이미 배송된 주문은..."
    }
  }

  return (
    <div>
      <h2>주문 #{order.id.value}</h2>
      <p>금액: {order.totalAmount.amount}원</p>
      <button onClick={handleCancel}>주문 취소</button>
    </div>
  )
}
```

### AI 활용 시 장점
```bash
# AI가 도메인 로직을 완벽히 이해
$ claude "Order Aggregate에 '부분 취소' 기능 추가해줘.
  단, 이미 배송 중인 아이템은 취소 불가"

# AI가 불변성 규칙을 지킴
$ claude "Money Value Object에 곱셈 메서드 추가해줘"
```

---

## 8. CQRS Pattern for Frontend State

백엔드의 CQRS(Command Query Responsibility Segregation)를 프론트엔드 상태 관리에 적용합니다.

### 핵심 개념
- **Command**: 상태를 변경하는 작업 (Create, Update, Delete)
- **Query**: 상태를 읽기만 하는 작업 (Read)
- **분리**: 읽기 모델과 쓰기 모델을 분리하여 최적화

### 구조 예시
```typescript
// state/product/queries.ts (읽기 전용)
import { useQuery } from '@tanstack/react-query'

export function useProductList() {
  return useQuery({
    queryKey: ['products'],
    queryFn: async () => {
      const res = await fetch('/api/products')
      return res.json()
    }
  })
}

export function useProductDetail(id: string) {
  return useQuery({
    queryKey: ['product', id],
    queryFn: async () => {
      const res = await fetch(`/api/products/${id}`)
      return res.json()
    }
  })
}

// state/product/commands.ts (쓰기 전용)
import { useMutation, useQueryClient } from '@tanstack/react-query'

export function useCreateProduct() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (data: CreateProductInput) => {
      const res = await fetch('/api/products', {
        method: 'POST',
        body: JSON.stringify(data)
      })
      return res.json()
    },
    onSuccess: () => {
      // Command 성공 시 Query 무효화
      queryClient.invalidateQueries({ queryKey: ['products'] })
    }
  })
}

export function useUpdateProduct() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async ({ id, data }: { id: string; data: UpdateProductInput }) => {
      const res = await fetch(`/api/products/${id}`, {
        method: 'PUT',
        body: JSON.stringify(data)
      })
      return res.json()
    },
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: ['product', variables.id] })
      queryClient.invalidateQueries({ queryKey: ['products'] })
    }
  })
}

// UI에서 사용
function ProductDetail({ id }: { id: string }) {
  const { data: product } = useProductDetail(id) // Query
  const updateProduct = useUpdateProduct() // Command

  const handleUpdate = (newData: UpdateProductInput) => {
    updateProduct.mutate({ id, data: newData })
  }

  return (
    <div>
      <h1>{product?.name}</h1>
      <button onClick={() => handleUpdate({ name: 'New Name' })}>
        수정
      </button>
    </div>
  )
}
```

### 고급: Optimistic Update + Event Sourcing
```typescript
export function useAddToCart() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (productId: string) => {
      return await cartApi.addItem(productId)
    },

    // Optimistic Update (즉시 UI 반영)
    onMutate: async (productId) => {
      await queryClient.cancelQueries({ queryKey: ['cart'] })

      const previousCart = queryClient.getQueryData(['cart'])

      queryClient.setQueryData(['cart'], (old: Cart) => ({
        ...old,
        items: [...old.items, { productId, quantity: 1 }]
      }))

      return { previousCart }
    },

    // 실패 시 롤백
    onError: (err, productId, context) => {
      queryClient.setQueryData(['cart'], context?.previousCart)
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] })
    }
  })
}
```

### AI 활용 시 장점
- **명확한 책임 분리**: AI가 "읽기 로직만 수정해줘" 요청에 정확히 대응
- **테스트 작성 용이**: AI가 Query와 Command를 독립적으로 테스트

---

## 9. Composable Architecture

SwiftUI의 TCA(The Composable Architecture)에서 영감을 받아 프론트엔드에 적용한 패턴입니다.

### 핵심 개념
- **단방향 데이터 흐름**: Redux와 유사하지만 더 타입 안전
- **Effect(부수효과) 격리**: 모든 비동기 작업을 명시적으로 관리
- **Reducer 조합**: 작은 Reducer들을 조합하여 큰 Reducer 생성

### 구조 예시
```typescript
// features/counter/counterSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

// State
type CounterState = {
  value: number
  status: 'idle' | 'loading' | 'succeeded' | 'failed'
}

// Actions (명시적 타입)
type CounterAction =
  | { type: 'counter/increment' }
  | { type: 'counter/decrement' }
  | { type: 'counter/incrementByAmount'; payload: number }

// Effect (비동기 작업)
export const incrementAsync = createAsyncThunk(
  'counter/incrementAsync',
  async (amount: number) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return amount
  }
)

// Reducer
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0, status: 'idle' } as CounterState,
  reducers: {
    increment: (state) => {
      state.value += 1
    },
    decrement: (state) => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(incrementAsync.pending, (state) => {
        state.status = 'loading'
      })
      .addCase(incrementAsync.fulfilled, (state, action) => {
        state.status = 'succeeded'
        state.value += action.payload
      })
  }
})

// UI
import { useAppDispatch, useAppSelector } from '@/store/hooks'

function Counter() {
  const count = useAppSelector((state) => state.counter.value)
  const status = useAppSelector((state) => state.counter.status)
  const dispatch = useAppDispatch()

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(counterSlice.actions.increment())}>
        +1
      </button>
      <button onClick={() => dispatch(incrementAsync(5))}>
        {status === 'loading' ? 'Loading...' : '+5 (Async)'}
      </button>
    </div>
  )
}
```

### AI와 함께 사용하는 고급 패턴
```typescript
// ✅ AI가 완벽하게 이해하는 Typed Reducer
type ProductState = {
  products: Product[]
  selectedProduct: Product | null
  filters: ProductFilters
  status: LoadingStatus
}

type ProductAction =
  | { type: 'products/loaded'; payload: Product[] }
  | { type: 'products/selected'; payload: string } // product ID
  | { type: 'products/filterChanged'; payload: Partial<ProductFilters> }
  | { type: 'products/clearFilters' }

function productReducer(
  state: ProductState,
  action: ProductAction
): ProductState {
  switch (action.type) {
    case 'products/loaded':
      return { ...state, products: action.payload, status: 'succeeded' }

    case 'products/selected':
      return {
        ...state,
        selectedProduct: state.products.find((p) => p.id === action.payload) || null
      }

    case 'products/filterChanged':
      return {
        ...state,
        filters: { ...state.filters, ...action.payload }
      }

    case 'products/clearFilters':
      return {
        ...state,
        filters: DEFAULT_FILTERS
      }

    default:
      return state
  }
}

// AI 활용 예시
// $ claude "productReducer에 '가격 범위 필터' 액션 추가해줘"
// → AI가 타입을 먼저 추가하고, reducer case도 자동 추가
```

---

## 요약: 무엇을 선택해야 할까?

### 프로젝트 규모별
-   **스타트업 / 빠른 MVP**: `Islands Architecture` (Next.js App Router) + `Vertical Slice`
-   **중규모 SaaS (10-50명)**: `Feature-Sliced Design (FSD)` + `CQRS`
-   **대규모 플랫폼 / 엔터프라이즈**: `FSD` + `DDD` + `Micro-Frontend`
-   **복잡한 금융/핀테크 로직**: `Clean Architecture` + `DDD`

### 팀 구성별
-   **단일 팀**: `FSD` 또는 `Vertical Slice`
-   **여러 팀 독립 운영**: `Micro-Frontend` + `Vertical Slice`
-   **도메인 전문가 협업**: `Domain-Driven Design`

### 기술 선택별
-   **Next.js 사용**: `Islands Architecture` + `Server Components`
-   **React (CSR)**: `FSD` + `Clean Architecture`
-   **상태 관리 복잡**: `CQRS` + `Composable Architecture`

### AI 활용도별
-   **AI 적극 활용**: 모든 아키텍처에 `AI-Native 원칙` 적용
-   **특히 중요**: Co-location, Explicit Typing, Small Functions

---

## 10. AI와 함께하는 아키텍처 Best Practices

### 실전 워크플로우

#### 1. 새 기능 개발 플로우
```bash
# 1단계: AI가 타입 정의
$ claude "사용자 알림 기능을 위한 타입을 정의해줘.
  Notification, NotificationType, NotificationPriority 포함"

# 2단계: AI가 폴더 구조 생성
$ claude "features/notifications 폴더에 FSD 구조 만들어줘"

# 3단계: README 작성 (직접 또는 AI와 협업)
$ claude "notifications 기능의 README.md를 작성해줘.
  역할, 제약사항, 에러 처리 포함"

# 4단계: AI가 구현
$ claude "README.md를 읽고 sendNotification 함수를 구현해줘.
  우선순위별 다른 채널 사용 (HIGH: SMS, MEDIUM: Push, LOW: Email)"

# 5단계: AI가 테스트
$ claude "sendNotification.ts의 모든 시나리오를 테스트해줘"

# 6단계: AI가 UI 컴포넌트
$ claude "NotificationList 컴포넌트를 만들어줘.
  읽지 않은 알림은 볼드 처리, 클릭 시 읽음 처리"
```

#### 2. 리팩토링 플로우
```bash
# 복잡한 컴포넌트 분리
$ claude "CheckoutPage.tsx를 읽고 200줄 이상이면
  작은 컴포넌트들로 분리해줘. FSD 원칙 준수"

# 타입 안전성 강화
$ claude "features/payment 폴더의 모든 any 타입을 찾아서
  명시적 타입으로 변경해줘"

# 도메인 로직 분리
$ claude "ProductCard.tsx에서 비즈니스 로직을 찾아서
  domain/product/ 폴더로 추출해줘"
```

#### 3. 디버깅 플로우
```bash
# 에러 원인 분석
$ claude "CartStore.ts에서 중복 추가 버그를 찾아줘.
  테스트 코드와 함께 확인"

# 성능 최적화
$ claude "ProductList.tsx에서 불필요한 리렌더링을 찾아서
  useMemo, useCallback으로 최적화해줘"
```

### AI와 함께 사용하는 코드 작성 규칙

#### 1. README-Driven Development
```markdown
<!-- features/checkout/README.md -->
# Checkout Feature

## Overview
사용자가 장바구니 상품을 구매하는 전체 프로세스를 담당합니다.

## User Flow
1. 장바구니 확인
2. 배송지 입력
3. 결제 수단 선택
4. 주문 확인
5. 결제 처리

## Business Rules
- 최소 주문 금액: 10,000원
- 배송비: 50,000원 이상 무료, 미만 3,000원
- 쿠폰은 1개만 적용 가능
- 품절 상품은 자동 제거

## Technical Constraints
- 결제 처리는 멱등성 보장 (같은 주문 ID 중복 결제 방지)
- 타임아웃: 결제 API 10초, 재고 확인 3초
- 재시도: 결제 실패 시 3회까지 재시도 (지수 백오프)

## Error Handling
- `CheckoutValidationError`: 장바구니가 비었거나 최소 금액 미만
- `OutOfStockError`: 품절 상품 포함
- `PaymentFailedError`: 결제 실패 (사유 포함)
- `InventoryReservationError`: 재고 확보 실패

## Dependencies
- `@/entities/product`: 상품 정보
- `@/entities/user`: 사용자 정보
- `@/features/cart`: 장바구니
- `@/shared/api/payment`: 결제 API

## API Endpoints
- `POST /api/checkout/validate`: 장바구니 검증
- `POST /api/checkout/reserve-inventory`: 재고 확보
- `POST /api/checkout/process-payment`: 결제 처리
```

이제 AI에게:
```bash
$ claude "features/checkout/README.md를 읽고 전체 기능을 구현해줘"
```

#### 2. Type-First Development
```typescript
// ✅ AI가 활용하기 좋은 타입 정의

// 1. Discriminated Union으로 상태 표현
type CheckoutState =
  | { status: 'idle' }
  | { status: 'validating' }
  | { status: 'validated'; data: ValidatedCart }
  | { status: 'processing-payment'; orderId: string }
  | { status: 'completed'; order: Order }
  | { status: 'failed'; error: CheckoutError }

// 2. Branded Types로 혼동 방지
type ProductID = string & { readonly __brand: 'ProductID' }
type OrderID = string & { readonly __brand: 'OrderID' }
type UserID = string & { readonly __brand: 'UserID' }

function processOrder(orderId: OrderID, userId: UserID) {
  // orderId와 userId를 바꿔 쓰면 컴파일 에러!
}

// 3. Exhaustive Check로 모든 케이스 처리 강제
function handleCheckoutState(state: CheckoutState) {
  switch (state.status) {
    case 'idle':
      return <IdleView />
    case 'validating':
      return <ValidatingView />
    case 'validated':
      return <PaymentView data={state.data} />
    case 'processing-payment':
      return <ProcessingView orderId={state.orderId} />
    case 'completed':
      return <SuccessView order={state.order} />
    case 'failed':
      return <ErrorView error={state.error} />
    default:
      // 새로운 status 추가 시 컴파일 에러 발생
      const exhaustiveCheck: never = state
      return exhaustiveCheck
  }
}
```

AI 활용:
```bash
$ claude "CheckoutState에 'refunding' 상태 추가해줘"
# → AI가 타입 정의, reducer, UI 핸들러까지 모두 업데이트
```

#### 3. Test-Driven with AI
```typescript
// 1. AI에게 테스트 먼저 작성하게 함
$ claude "calculateShippingFee 함수의 요구사항:
  - 50,000원 이상: 무료
  - 30,000원 이상: 2,000원
  - 미만: 3,000원
  - 제주도: +3,000원
  이 요구사항의 테스트를 먼저 작성해줘"

// AI가 생성한 테스트
describe('calculateShippingFee', () => {
  it('should return 0 for orders over 50,000 won', () => {
    expect(calculateShippingFee(50000, 'seoul')).toBe(0)
    expect(calculateShippingFee(60000, 'seoul')).toBe(0)
  })

  it('should return 2,000 for orders between 30,000 and 50,000 won', () => {
    expect(calculateShippingFee(30000, 'seoul')).toBe(2000)
    expect(calculateShippingFee(49999, 'seoul')).toBe(2000)
  })

  it('should return 3,000 for orders under 30,000 won', () => {
    expect(calculateShippingFee(29999, 'seoul')).toBe(3000)
    expect(calculateShippingFee(10000, 'seoul')).toBe(3000)
  })

  it('should add 3,000 won for Jeju Island', () => {
    expect(calculateShippingFee(60000, 'jeju')).toBe(3000)
    expect(calculateShippingFee(40000, 'jeju')).toBe(5000) // 2000 + 3000
    expect(calculateShippingFee(20000, 'jeju')).toBe(6000) // 3000 + 3000
  })

  it('should handle edge cases', () => {
    expect(calculateShippingFee(0, 'seoul')).toBe(3000)
    expect(calculateShippingFee(-100, 'seoul')).toBe(3000)
  })
})

// 2. 이제 AI에게 구현하게 함
$ claude "위 테스트를 통과하는 calculateShippingFee 함수를 구현해줘"
```

#### 4. Incremental Refactoring
```bash
# 단계적 리팩토링 - AI가 안전하게 수행 가능

# Step 1: 타입 추가
$ claude "ProductCard.tsx의 모든 props에 명시적 타입 추가해줘"

# Step 2: 로직 분리
$ claude "ProductCard.tsx에서 가격 계산 로직을 찾아서
  calculateProductPrice.ts로 분리해줘. 테스트도 함께"

# Step 3: 컴포넌트 분리
$ claude "ProductCard.tsx가 50줄 이상이면
  ProductImage, ProductInfo, ProductActions로 분리해줘"

# Step 4: 성능 최적화
$ claude "분리된 컴포넌트들을 React.memo로 최적화해줘.
  불필요한 리렌더링 방지"
```

### AI 코드 리뷰 활용
```bash
# 1. 보안 취약점 검사
$ claude "features/auth 폴더의 모든 파일을 검사해서
  보안 취약점(XSS, CSRF, 평문 저장 등) 찾아줘"

# 2. 성능 이슈 검사
$ claude "ProductList.tsx를 분석해서 성능 문제
  (불필요한 리렌더링, 메모리 누수 등) 찾아줘"

# 3. 아키텍처 위반 검사
$ claude "features/cart 폴더가 FSD 원칙을 위반하는지 확인해줘.
  특히 상위 레이어 import 여부 체크"

# 4. 접근성 검사
$ claude "LoginForm.tsx의 접근성 문제
  (ARIA 속성, 키보드 네비게이션 등) 찾아줘"
```

### AI 페어 프로그래밍 패턴

#### 패턴 1: AI가 초안 작성 → 사람이 리뷰
```bash
$ claude "장바구니에서 쿠폰 적용 기능을 구현해줘.
  1개만 적용 가능하고, 중복 적용 시 에러"

# AI 구현 후 사람이 비즈니스 로직 검증
# "음, VIP는 2개 적용 가능해야 하는데..."

$ claude "VIP 회원은 쿠폰 2개 적용 가능하도록 수정해줘"
```

#### 패턴 2: 사람이 인터페이스 정의 → AI가 구현
```typescript
// 사람이 작성
interface PaymentGateway {
  /**
   * 결제를 처리합니다.
   * @throws PaymentFailedError 결제 실패 시
   * @throws InvalidCardError 잘못된 카드 정보
   * @throws InsufficientFundsError 잔액 부족
   */
  processPayment(request: PaymentRequest): Promise<PaymentResult>

  /**
   * 결제를 취소합니다. 멱등성 보장.
   */
  cancelPayment(paymentId: string): Promise<void>

  /**
   * 결제 상태를 조회합니다.
   */
  getPaymentStatus(paymentId: string): Promise<PaymentStatus>
}
```

```bash
$ claude "PaymentGateway 인터페이스를 구현한
  TossPaymentGateway 클래스를 만들어줘.
  Toss Payments API 사용"
```

#### 패턴 3: 사람이 테스트 작성 → AI가 구현
```typescript
// 사람이 작성 (비즈니스 요구사항을 테스트로 표현)
describe('VIP 회원 혜택', () => {
  it('VIP는 쿠폰 2개 동시 적용 가능', () => {
    const user = createVIPUser()
    const coupons = [coupon10000, coupon5000]
    const result = applyCoupons(user, 50000, coupons)
    expect(result.discount).toBe(15000)
  })

  it('일반 회원은 쿠폰 1개만 적용', () => {
    const user = createStandardUser()
    const coupons = [coupon10000, coupon5000]
    expect(() => applyCoupons(user, 50000, coupons))
      .toThrow('일반 회원은 쿠폰을 1개만 사용할 수 있습니다')
  })
})
```

```bash
$ claude "위 테스트를 통과하는 applyCoupons 함수를 구현해줘"
```

---

## 실전 프로젝트 구조 예시

### E-commerce 프로젝트 (FSD + AI-Native)
```
src/
  app/
    providers/
      AuthProvider.tsx
      ThemeProvider.tsx
      QueryProvider.tsx
    layout.tsx
    globals.css

  pages/
    home/
      ui/HomePage.tsx
      README.md                   # AI: "홈페이지 요구사항"
    product-detail/
      ui/ProductDetailPage.tsx
      model/useProductDetail.ts
      README.md
    checkout/
      ui/CheckoutPage.tsx
      model/useCheckout.ts
      README.md                   # 가장 중요한 비즈니스 로직 설명

  widgets/
    header/
      ui/Header.tsx
      model/useHeader.ts
    product-card/
      ui/ProductCard.tsx
      ui/ProductCard.stories.tsx  # Storybook
      README.md                   # AI: "ProductCard 사용법"

  features/
    add-to-cart/
      ui/AddToCartButton.tsx
      api/addToCartMutation.ts
      model/useAddToCart.ts
      model/useAddToCart.spec.ts
      README.md
    apply-coupon/
      ui/CouponInput.tsx
      domain/applyCoupon.ts       # 순수 함수
      domain/applyCoupon.spec.ts
      README.md

  entities/
    product/
      ui/ProductImage.tsx
      model/product.types.ts      # Branded Types 사용
      model/product.schema.ts     # Zod 스키마
      api/productApi.ts
      README.md                   # AI: "Product 엔티티 설명"
    user/
      model/user.types.ts
      model/user.permissions.ts
      api/userApi.ts

  shared/
    ui/
      button/
        Button.tsx
        Button.stories.tsx
        Button.spec.tsx
        README.md                 # AI: "Button 컴포넌트 사용법"
      input/
      modal/
    lib/
      validation/
        validators.ts
        validators.spec.ts
      formatting/
        formatCurrency.ts
        formatCurrency.spec.ts
        README.md
    api/
      client.ts                   # Axios 설정
      error-handler.ts
    config/
      constants.ts
      env.ts

  domain/                         # Clean Architecture 도메인 로직
    order/
      Order.ts                    # Aggregate
      OrderItem.ts
      OrderStatus.ts
      OrderCalculator.ts          # 도메인 서비스
      OrderCalculator.spec.ts
      README.md                   # AI: "주문 도메인 규칙"
    payment/
      Payment.ts
      PaymentGateway.ts           # Interface
      TossPaymentGateway.ts       # Implementation
      README.md
```

---

## 마무리: AI 시대의 프론트엔드 아키텍처 핵심

### 3대 원칙
1. **Co-location**: 관련 코드를 한 곳에 모아 AI의 컨텍스트 효율성 극대화
2. **Explicit Typing**: 타입으로 AI에게 명확한 제약 조건 제공
3. **Small Functions**: AI가 한 번에 이해하고 수정할 수 있는 크기로 유지

### AI 활용을 위한 체크리스트
- [ ] 모든 기능 폴더에 README.md 작성
- [ ] `any` 타입 사용 금지, 모든 타입 명시
- [ ] 함수는 20줄 이하로 유지
- [ ] 비즈니스 로직과 UI 로직 분리
- [ ] 모든 도메인 로직에 테스트 작성
- [ ] 변수/함수명은 서술적으로 (약어 지양)
- [ ] 복잡한 조건문은 함수로 추출 (`isEligibleForDiscount()`)

### 프로젝트 시작 템플릿
```bash
# 1. 기술 스택 선택
$ claude "Next.js 15 + TypeScript + Tailwind + Zustand로
  프로젝트 초기 설정해줘. FSD 구조 적용"

# 2. 아키텍처 문서 생성
$ claude "이 프로젝트의 아키텍처 가이드를 ARCHITECTURE.md에 작성해줘.
  FSD 레이어 규칙, 폴더 구조, 네이밍 컨벤션 포함"

# 3. AI 협업 규칙 생성
$ claude ".cursorrules 파일에 AI 협업 규칙을 작성해줘.
  - 모든 파일은 README.md 먼저 읽기
  - any 타입 금지
  - 함수는 20줄 이하
  - 테스트 필수 작성"

# 4. 첫 기능 개발
$ claude "features/auth/login 기능을 구현해줘.
  README 작성부터 시작"
```
