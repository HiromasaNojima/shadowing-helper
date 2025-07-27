# アーキテクチャ図：英語シャドーイング練習ツール

## システム全体図

```mermaid
graph TB
    subgraph "Client (Browser)"
        UI[UI Layer<br/>Next.js + React]
        Store[State Management<br/>Zustand]
        API[APIs]
        
        subgraph "Components"
            YT[YouTube Player<br/>Component]
            REC[Audio Recorder<br/>Component]
            SEC[Section Control<br/>Component]
        end
        
        subgraph "Hooks"
            H1[useYouTube]
            H2[useRecorder]
            H3[useSection]
        end
        
        subgraph "External APIs"
            YTAPI[YouTube IFrame API]
            MEDIA[MediaRecorder API]
        end
    end
    
    subgraph "Storage"
        LS[LocalStorage<br/>Settings]
        IDB[IndexedDB<br/>Recordings]
    end
    
    UI --> Store
    UI --> Components
    Components --> Hooks
    Hooks --> Store
    YT --> H1
    REC --> H2
    SEC --> H3
    H1 --> YTAPI
    H2 --> MEDIA
    Store --> LS
    H2 --> IDB
```

## データフロー図

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant YouTubePlayer
    participant AudioRecorder
    participant Store
    participant Storage
    
    User->>UI: YouTube URLを入力
    UI->>Store: URLを保存
    Store->>YouTubePlayer: 動画IDを渡す
    YouTubePlayer->>YouTubePlayer: 動画を読み込み
    
    User->>UI: 区間を設定
    UI->>Store: 開始/終了時間を保存
    
    User->>UI: 録音開始
    UI->>AudioRecorder: 録音開始
    AudioRecorder->>AudioRecorder: マイクアクセス許可
    AudioRecorder->>AudioRecorder: 録音中...
    
    User->>UI: 録音停止
    AudioRecorder->>Store: 録音データを保存
    Store->>Storage: IndexedDBに保存
    
    User->>UI: 録音再生
    UI->>Storage: 録音データ取得
    Storage->>UI: 録音データ返却
    UI->>User: 音声再生
```

## コンポーネント関連図

```mermaid
graph LR
    subgraph "Pages"
        Home[Home Page]
    end
    
    subgraph "Layout Components"
        Layout[Root Layout]
        Header[Header]
    end
    
    subgraph "Feature Components"
        YTInput[YouTube URL Input]
        YTPlayer[YouTube Player]
        Controls[Player Controls]
        Section[Section Settings]
        Recorder[Audio Recorder]
        RecList[Recording List]
    end
    
    subgraph "UI Components (shadcn/ui)"
        Button[Button]
        Input[Input]
        Slider[Slider]
        Card[Card]
    end
    
    Home --> YTInput
    Home --> YTPlayer
    Home --> Controls
    Home --> Section
    Home --> Recorder
    Home --> RecList
    
    YTInput --> Input
    Controls --> Button
    Section --> Slider
    Section --> Input
    Recorder --> Button
    RecList --> Card
    RecList --> Button
```

## 状態管理フロー

```mermaid
stateDiagram-v2
    [*] --> Idle
    
    Idle --> VideoLoaded: URL入力
    VideoLoaded --> Playing: 再生開始
    VideoLoaded --> SectionSet: 区間設定
    
    Playing --> Paused: 一時停止
    Paused --> Playing: 再開
    Playing --> Looping: ループ有効
    
    SectionSet --> Playing: 再生開始
    
    state Recording {
        [*] --> Ready
        Ready --> Recording: 録音開始
        Recording --> Stopped: 録音停止
        Stopped --> Saved: 保存
        Saved --> Ready
    }
    
    Playing --> Recording: 録音開始
    Looping --> Recording: 録音開始
```

## 技術スタック層

```mermaid
graph TD
    subgraph "Frontend"
        A[Next.js 15.4.1<br/>App Router]
        B[React 19]
        C[TypeScript 5.x]
    end
    
    subgraph "Styling"
        D[Tailwind CSS 3.x]
        E[shadcn/ui]
    end
    
    subgraph "State Management"
        F[Zustand 5.x]
    end
    
    subgraph "APIs"
        G[YouTube IFrame API]
        H[MediaRecorder API]
        I[Web Storage API]
    end
    
    subgraph "Build Tools"
        J[Turbopack]
        K[PostCSS]
        L[ESLint]
        M[Prettier]
    end
    
    A --> B
    A --> C
    B --> E
    E --> D
    B --> F
    A --> G
    A --> H
    F --> I
    A --> J
    D --> K
    C --> L
    C --> M
```

## フォルダ構造詳細

```
shadowing-helper/
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── layout.tsx           # ルートレイアウト
│   │   ├── page.tsx             # ホームページ
│   │   ├── globals.css          # グローバルスタイル
│   │   └── favicon.ico          
│   │
│   ├── components/              # コンポーネント
│   │   ├── ui/                  # shadcn/ui 基本コンポーネント
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── slider.tsx
│   │   │   └── card.tsx
│   │   │
│   │   ├── youtube-player/      # YouTube再生機能
│   │   │   ├── index.tsx
│   │   │   ├── player.tsx
│   │   │   └── url-input.tsx
│   │   │
│   │   ├── audio-recorder/      # 録音機能
│   │   │   ├── index.tsx
│   │   │   ├── recorder.tsx
│   │   │   ├── recording-list.tsx
│   │   │   └── audio-player.tsx
│   │   │
│   │   └── section-control/     # 区間設定機能
│   │       ├── index.tsx
│   │       ├── time-input.tsx
│   │       └── loop-control.tsx
│   │
│   ├── hooks/                   # カスタムフック
│   │   ├── use-youtube.ts       # YouTube API管理
│   │   ├── use-recorder.ts      # 録音機能管理
│   │   ├── use-section.ts       # 区間設定管理
│   │   └── use-storage.ts       # ストレージ管理
│   │
│   ├── lib/                     # ユーティリティ
│   │   ├── utils.ts             # 共通関数
│   │   ├── youtube.ts           # YouTube関連
│   │   ├── audio.ts             # 音声処理関連
│   │   └── storage.ts           # ストレージ関連
│   │
│   ├── stores/                  # Zustand ストア
│   │   ├── player-store.ts      # 再生状態
│   │   ├── recorder-store.ts    # 録音状態
│   │   └── section-store.ts     # 区間設定状態
│   │
│   └── types/                   # 型定義
│       ├── index.ts             # 共通型
│       ├── youtube.d.ts         # YouTube API型
│       └── recorder.d.ts        # 録音関連型
│
├── public/                      # 静的ファイル
│   └── icons/                   # アイコン
│
├── docs/                        # ドキュメント
│   ├── requirements.md          # 要件定義書
│   ├── technology-stack.md      # 技術選定書
│   └── architecture-diagram.md  # アーキテクチャ図
│
├── .env.local                   # 環境変数
├── .gitignore
├── next.config.js               # Next.js設定
├── tailwind.config.ts           # Tailwind設定
├── tsconfig.json                # TypeScript設定
├── package.json
└── README.md
```

## セキュリティとプライバシー

```mermaid
graph LR
    subgraph "Security Measures"
        A[HTTPS Only]
        B[CSP Headers]
        C[Input Validation]
        D[Local Storage Only]
    end
    
    subgraph "Privacy"
        E[No Server Upload]
        F[Local Processing]
        G[User Consent]
        H[Data Deletion]
    end
    
    A --> G
    B --> C
    D --> E
    F --> H
```

この設計により、シンプルかつ拡張可能なアーキテクチャを実現し、ユーザーのプライバシーを保護しながら高品質なシャドーイング練習体験を提供します。