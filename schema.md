# Berber GraphQL Şeması

Bu doküman, Berber uygulamasının GraphQL şemasını detaylı olarak açıklar.

## İçindekiler

- [Temel Tipler](#temel-tipler)
- [Direktifler](#direktifler)
- [Skalerler](#skalerler)
- [Enum Tipleri](#enum-tipleri)
- [Arayüzler](#arayüzler)
- [Ana Tipler](#ana-tipler)
- [Bağlantı Tipleri](#bağlantı-tipleri)
- [Giriş (Input) Tipleri](#giriş-tipleri)
- [Sorgular (Queries)](#sorgular)
- [Mutasyonlar (Mutations)](#mutasyonlar)

## Temel Tipler

### Node

Tüm tipler için temel arayüz.

```graphql
interface Node {
  id: ID!
}
```

## Direktifler

### @resolver

```graphql
directive @resolver(permissions: [String!]!) on FIELD_DEFINITION
```

Bu direktif, alanın bir çözümleyici gerektirdiğini işaretler. İzinler boşsa, alanın herkese açık olduğu anlamına gelir. Verilen izinler VEYA mantığıyla kontrol edilir.

### @from

```graphql
directive @from(collection: String!) on FIELD_DEFINITION
```

Bu direktif, alanın belirtilen koleksiyondan bir belge olduğunu işaretler.

### @domain

```graphql
directive @domain(domain: Domains!) on FIELD_DEFINITION
```

Alan için etki alanını belirtir.

## Skalerler

- `DateTime`: Tarih ve zaman değerleri için
- `JSON`: JSON formatında veriler için
- `Any`: Herhangi bir tip için
- `Email`: E-posta adresleri için
- `PhoneNumber`: Telefon numaraları için
- `Contact`: İletişim bilgileri için
- `VerifyCode`: Doğrulama kodları için
- `Base64Data`: Base64 kodlanmış veriler için
- `Avatar`: HSL veya ID değerleri için
- `Hsl`: HSL renk değerleri için

## Enum Tipleri

### Domains

```graphql
enum Domains {
  user
  org
  admin
  public
  auth
}
```

### OrganizationType

```graphql
enum OrganizationType {
  MULTI_BRANCH
  SINGLE_BRANCH
  FREELANCER
}
```

### BranchType

```graphql
enum BranchType {
  FRANCHISE
  COMPANY_OWNED
}
```

### MemberStatus

```graphql
enum MemberStatus {
  ACTIVE
  PENDING
  INACTIVE
}
```

### ServicerStatus

```graphql
enum ServicerStatus {
  ACTIVE
  INACTIVE
}
```

### PaymentStatus

```graphql
enum PaymentStatus {
  PENDING
  PAID
  CANCELLED
  REFUNDED
}
```

### RefundStatus

```graphql
enum RefundStatus {
  PENDING
  REFUNDED
}
```

## Ana Tipler

### User

```graphql
type User implements Node {
  id: ID!
  name: String!
  avatar: Avatar!
  createdAt: DateTime!
  updatedAt: DateTime!
  phone: PhoneNumber
  email: Email
  organization: Organization
  organization_ID: ID
  createdBy_ID: ID
  createdBy: Member
  authMethods(pagination: PaginationInput): AuthMethodConnection!
  members(pagination: PaginationInput): MemberConnection!
  reviews(pagination: PaginationInput): ReviewConnection!
}
```

### Organization

```graphql
type Organization implements Node {
  id: ID!
  name: String!
  type: OrganizationType!
  avatar: Avatar!
  address: Address!
  location: Location!
  active_times: [ActiveTime!]!
  booking_buffer: Int!
  booking_range: Int!
  members(pagination: PaginationInput): MemberConnection!
  branches(pagination: PaginationInput): BranchConnection!
  services(pagination: PaginationInput): ServiceConnection!
  servicers(pagination: PaginationInput): ServicerConnection!
  reviews(pagination: PaginationInput): ReviewConnection!
}
```

### Branch

```graphql
type Branch implements Node {
  id: ID!
  name: String!
  type: BranchType!
  organization: Organization!
  organization_ID: ID!
  address: Address!
  location: Location!
  active_times: [ActiveTime!]!
  booking_buffer: Int!
  booking_range: Int!
  members(pagination: PaginationInput): MemberConnection!
  services(pagination: PaginationInput): ServiceConnection!
  servicers(pagination: PaginationInput): ServicerConnection!
}
```

### Service

```graphql
type Service implements Node {
  id: ID!
  serviceType: ServiceType!
  serviceType_ID: ID!
  organization: Organization!
  organization_ID: ID!
  duration: Int!
  price: Float!
  servicer: Servicer!
  servicer_ID: ID!
  branch: Branch!
  branch_ID: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Servicer

```graphql
type Servicer implements Node {
  id: ID!
  member: Member!
  member_ID: ID!
  branch: Branch!
  branch_ID: ID!
  organization: Organization!
  organization_ID: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  services(pagination: PaginationInput): ServiceConnection!
  availabilities(pagination: PaginationInput): AvailabilityConnection!
  reviews(pagination: PaginationInput): ReviewConnection!
}
```

### Review

```graphql
type Review implements Node {
  id: ID!
  rating: Int!
  comment: String!
  media: [ReviewMedia!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

## Bağlantı (Connection) Tipleri

GraphQL'de sayfalama ve filtreleme için Connection pattern kullanılmaktadır. Her Connection tipi şu yapıyı takip eder:

```graphql
interface Connection {
  pageInfo: PageInfo!
  items: [Node!]!
  groups: GroupInfo
}

type PageInfo {
  hasNextPage: Boolean!
  nextCursor: String
}

type GroupInfo {
  totalCount: Int!
  groups: [Group!]!
}
```

Örnek kullanım:

```graphql
# Bir organizasyonun üyelerini sayfalı şekilde almak için:
organization {
  members(pagination: { limit: 10, cursor: "xyz" }) {
    items {
      id
      name
    }
    pageInfo {
      hasNextPage
      nextCursor
    }
  }
}
```

## Ana Tipler (devam)

### Member

```graphql
type Member implements Node {
  id: ID!
  user: User
  user_ID: ID
  contact: Contact
  status: MemberStatus!
  role: Role!
  role_ID: ID!
  branch: Branch
  branch_ID: ID
  organization: Organization!
  organization_ID: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### AuthMethod

```graphql
type AuthMethod implements Node {
  id: ID!
  provider: AuthProvider!
  user: User!
  user_ID: ID!
  info: ProviderInfo!
  verified: Boolean!
}

type ProviderInfo {
  id: ID!
  name: String!
  email: String!
  avatar: String
  token: String
  refreshToken: String
  expiresAt: DateTime
}
```

### Booking

```graphql
type Booking implements Node {
  id: ID!
  user: User!
  user_ID: ID!
  servicer: Servicer!
  servicer_ID: ID!
  service: Service!
  service_ID: ID!
  branch: Branch!
  branch_ID: ID!
  organization: Organization!
  organization_ID: ID!
  status: BookingStatus!
  start: DateTime!
  end: DateTime!
  duration: Int!
  price: Float!
  payment: Payment
  payment_ID: ID
  refund: Refund
  refund_ID: ID
  review: Review
  review_ID: ID
  cancellation: Cancellation
  cancellation_ID: ID
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### Payment

```graphql
type Payment implements Node {
  id: ID!
  amount: Float!
  currency: String!
  status: PaymentStatus!
  booking: Booking!
  booking_ID: ID!
  organization: Organization!
  organization_ID: ID!
  servicer: Servicer!
  servicer_ID: ID!
  branch: Branch!
  branch_ID: ID!
  user: User
  user_ID: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  refund: Refund
  refund_ID: ID
}
```

### Role

```graphql
type Role implements Node {
  id: ID!
  name: String!
  permissions: [String!]!
  organization: Organization!
  organization_ID: ID!
  members(pagination: PaginationInput): MemberConnection!
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

### ServiceType

```graphql
type ServiceType implements Node {
  id: ID!
  name: String!
  avatar: String!
  booking_count(range: DateRangeInput, verified: Boolean): Int!
  servicer_count: Int!
  createdAt: DateTime!
  updatedAt: DateTime!
  createdBy: Admin!
  createdBy_ID: ID!
  updatedBy: Admin!
  updatedBy_ID: ID!
}
```

### Availability

```graphql
type Availability implements Node {
  id: ID!
  start: DateTime!
  end: DateTime!
  duration: Int!
  servicer: Servicer!
  servicer_ID: ID!
}
```

### CancellationPolicy

```graphql
type CancellationPolicy implements Node {
  id: ID!
  ranges: [CancellationPolicyRange!]!
  organization: Organization!
  organization_ID: ID!
  createdAt: DateTime!
  updatedAt: DateTime!
  branch: Branch
  branch_ID: ID
}
```

## Sorgular

### Ana Sorgu Tipleri

```graphql
type Query {
  admin: AdminQuery!
  auth: AuthQuery!
  org: OrgQuery!
  public: PublicQuery!
  user: UserQuery!
}
```

### PublicQuery

```graphql
type PublicQuery {
  user(id: ID!): User!
  service(id: ID!): Service!
  servicer(id: ID!): Servicer!
  branch(id: ID!): Branch!
  service_type(id: ID!): ServiceType!
  organization(id: ID!): Organization!
  supported_languages: [SupportedLanguage!]!
  language_document(language: String!): LanguageDocument!
}
```

### UserQuery

```graphql
type UserQuery {
  home: HomeResponse!
  booking(id: ID!): Booking!
}
```

## Mutasyonlar

### Ana Mutasyon Tipleri

```graphql
type Mutation {
  admin: AdminMutation!
  auth: AuthMutation!
  org: OrgMutation!
  user: UserMutation!
  public: PublicMutation!
}
```

### UserMutation

```graphql
type UserMutation {
  update_profile(input: UpdateProfileRequest!): User!
  make_booking(input: MakeBookingRequest!): Booking!
  cancel_booking(id: ID!): Boolean!
  review_booking(id: ID!, input: ReviewBookingRequest!): Boolean!
  remove_review(id: ID!): Boolean!
  add_favorite(id: ID!): Boolean!
  remove_favorite(id: ID!): Boolean!
  add_to_cart(id: ID!): Boolean!
  remove_from_cart(id: ID!): Boolean!
  clear_cart: Boolean!
  checkout: Boolean!
  request_refund(id: ID!): Boolean!
}
```

### AuthMutation

```graphql
type AuthMutation {
  login_email(input: LoginEmailRequest!): AuthResponse!
  login_phone(input: LoginPhoneRequest!): AuthResponse!
  login_google(input: LoginGoogleRequest!): AuthResponse!
  login_apple(input: LoginAppleRequest!): AuthResponse!
  logout: Boolean!
  reset_password(input: ResetPasswordRequest!): Boolean!
  verify_forgot_password(
    input: VerifyForgotPasswordRequest!
  ): VerifyForgotPasswordResponse!
  change_password(input: ChangePasswordRequest!): AuthResponse!
  resend_verify_email(input: ResendVerifyEmailRequest!): Boolean!
}
```
