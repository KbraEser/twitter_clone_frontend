# Twitter Clone UI

X (Twitter) benzeri bir sosyal medya uygulamasının React + TypeScript ile yazılmış ön yüzü (frontend). Arayüz koyu temalıdır ve Türkçedir. Veriler ayrı bir REST backend'inden gelir (bu repoda yalnızca UI vardır).

## Özellikler

- **Kimlik doğrulama:** Kayıt ol (ad, soyad, e-posta, şifre, opsiyonel profil resmi URL'si) ve giriş yap. JWT `localStorage`'da saklanır, oturum sayfa yenilense de korunur.
- **Rota koruma:** Giriş yapmamış kullanıcılar `/login`'e, giriş yapmış kullanıcılar `/login` ve `/register` yerine `/`'e yönlendirilir.
- **Ana sayfa:** Tüm tweet'lerin akışı ve tweet oluşturma kutusu.
- **Tweet işlemleri:** Oluşturma, düzenleme, silme (düzenleme/silme yalnızca tweet sahibine açık).
- **Retweet ve alıntı tweet:** Basit retweet ya da kendi yorumunla alıntı (quote) tweet.
- **Beğeni:** Tweet ve yorumlar beğenilebilir / beğenisi geri alınabilir.
- **Yorumlar:** Tweet detay sayfasında ve kartta yorum yazma, düzenleme, silme.
- **Profil sayfası:** `/profile/:userId` ile herhangi bir kullanıcının tweet'leri.
- **Tweet detay sayfası:** `/tweet/:id`.

## Teknolojiler

| Alan | Araç |
| --- | --- |
| Framework | React 19 |
| Dil | TypeScript |
| Build | Vite |
| Stil | Tailwind CSS 4 (`@tailwindcss/vite`) |
| State | Redux Toolkit + react-redux |
| Routing | React Router 7 |
| HTTP | Axios |
| Lint | ESLint (typescript-eslint, react-hooks, react-refresh) |

## Başlangıç

### Gereksinimler

- Node.js (Vite 8 için güncel bir LTS sürümü)
- Çalışan bir backend: `http://localhost:3000`

### Kurulum ve çalıştırma

```bash
npm install
npm run dev
```

Uygulama Vite'in varsayılan adresinde açılır (genellikle `http://localhost:5173`).

### Scriptler

| Komut | Açıklama |
| --- | --- |
| `npm run dev` | Geliştirme sunucusunu başlatır |
| `npm run build` | Tip kontrolü (`tsc -b`) ve production build (`dist/`) |
| `npm run preview` | Production build'i yerelde önizler |
| `npm run lint` | ESLint çalıştırır |

## Backend bağlantısı

Axios istemcisi `baseURL: '/api'` kullanır. Geliştirme sırasında Vite bu istekleri proxy'ler (`vite.config.ts`):

```
/api/*  →  http://localhost:3000/*   (/api öneki kaldırılır)
```

Backend farklı bir adreste çalışıyorsa `vite.config.ts` içindeki `target` değerini değiştirin. Production'da `/api` isteklerini backend'e yönlendirmek için bir reverse proxy gerekir.

Beklenen endpoint'ler:

| Metot | Yol | Amaç |
| --- | --- | --- |
| POST | `/auth/login`, `/auth/register` | Giriş / kayıt |
| GET | `/tweet/findAll` | Tüm tweet'ler |
| GET | `/tweet/findByUserId?id=` | Kullanıcının tweet'leri |
| GET | `/tweet/findById?id=` | Tek tweet |
| POST | `/tweet` | Tweet / retweet / alıntı oluştur (`content`, `tweetId`) |
| PUT / DELETE | `/tweet/:id` | Güncelle / sil |
| GET | `/comment/findByTweetId?id=` | Tweet'in yorumları |
| POST | `/comment` | Yorum ekle |
| PUT / DELETE | `/comment/:id` | Yorum güncelle / sil (silmede `userId` query) |
| POST | `/like`, `/dislike` | Tweet ya da yorumu beğen / beğeniyi kaldır |

Her istekte `Authorization: Bearer <token>` başlığı eklenir. Herhangi bir yanıt `401` dönerse kullanıcı çıkış yaptırılır ve `/login`'e yönlendirilir.

## Proje yapısı

```
src/
├── main.tsx              Giriş noktası (Redux Provider)
├── App.tsx               Rotalar, GuestRoute / ProtectedRoute
├── index.css             Tailwind ve tema renkleri (twitter-*)
├── api/
│   ├── client.ts         Axios instance, token interceptor'ları, localStorage auth
│   ├── auth.ts           login, register
│   ├── tweets.ts         Tweet CRUD ve sorgular
│   ├── comments.ts       Yorum CRUD
│   └── likes.ts          like / dislike
├── store/
│   ├── index.ts          Store yapılandırması
│   ├── hooks.ts          Tipli useAppDispatch / useAppSelector
│   ├── authSlice.ts      Oturum durumu, loginUser thunk'ı, logout
│   ├── tweetSlice.ts     Tweet listesi, fetchTweets thunk'ı
│   └── likesSlice.ts     Beğenilen tweet/yorum ID'leri
├── components/
│   ├── Layout.tsx        Sidebar + içerik alanı
│   ├── Sidebar.tsx       Navigasyon, kullanıcı bilgisi, çıkış
│   ├── ProtectedRoute.tsx
│   ├── ComposeTweet.tsx  Tweet yazma formu
│   ├── TweetCard.tsx     Tweet kartı (beğeni, retweet, yorum, düzenle, sil)
│   └── CommentItem.tsx   Tek yorum
├── pages/                Login, Register, Home, Profile, TweetDetail
├── types/index.ts        Paylaşılan TypeScript tipleri (User, Tweet, Comment, istek/yanıt)
└── utils/likesStorage.ts Beğenileri kullanıcı bazında localStorage'a yazar, hata mesajı yardımcıları
```

## Mimari notlar

- **Oturum:** `{ token, userId, email, name, surname }` `twitter_auth` anahtarıyla `localStorage`'da tutulur; başlangıç durumu `authSlice` tarafından buradan okunur.
- **Beğeniler:** Backend tweet'in kullanıcı tarafından beğenilip beğenilmediğini döndürmediği için beğeni durumu istemci tarafında `twitter_liked_<userId>` anahtarıyla `localStorage`'da tutulur. Backend "already liked" ya da "like not found" hatası verdiğinde durum otomatik olarak senkronlanır.
- **Retweet modeli:** Retweet ve alıntılar `parentTweet` alanı olan normal tweet'lerdir. İçeriği boş olan ve `parentTweet`'i bulunan tweet "saf retweet" sayılır; düzenlenemez.
- **Tüm tweet'ler fallback'i:** `/tweet/findAll` `404` veya `405` dönerse, kullanıcı ID'leri 1–100 için `findByUserId` çağrılıp sonuçlar birleştirilir. Bu geçici bir çözümdür; backend'de `findAll` varsa devreye girmez.
- **Tema:** Renkler `src/index.css` içindeki `@theme` bloğunda tanımlıdır (`twitter-blue`, `twitter-border` vb.).

