# 技術選定書：英語シャドーイング練習ツール

## 1. 技術スタック概要

### フロントエンドフレームワーク
- **Next.js v15.4.1** (最新版)
  - App Routerを使用
  - Server Componentsによる高速な初期レンダリング
  - 静的サイト生成（SSG）でホスティングコストを削減

### スタイリング
- **Tailwind CSS v3.x**
  - ユーティリティファーストCSS
  - レスポンシブデザインが容易
  - カスタマイズ性が高い

### UIコンポーネント
- **shadcn/ui**
  - 美しくアクセシブルなコンポーネント
  - カスタマイズ可能
  - Radix UIベースで堅牢

### 状態管理
- **Zustand v5.x**
  - 軽量で学習コストが低い
  - TypeScriptサポートが優秀
  - DevTools対応

### 音声録音・処理
- **MediaRecorder API** (標準Web API)
  - ブラウザネイティブの録音機能
  - 追加ライブラリ不要
  - WebM/Opus形式で録音

### 動画再生
- **YouTube IFrame Player API**
  - YouTube公式API
  - 再生制御が豊富
  - 区間リピート機能の実装が可能

### 型安全性
- **TypeScript v5.x**
  - 型安全性の確保
  - 開発効率の向上
  - エラーの早期発見

## 2. アーキテクチャ設計

### ディレクトリ構成
```
shadowing-helper/
├── src/
│   ├── app/                  # Next.js App Router
│   │   ├── layout.tsx        # ルートレイアウト
│   │   ├── page.tsx          # ホームページ
│   │   └── globals.css       # グローバルスタイル
│   ├── components/           # UIコンポーネント
│   │   ├── ui/              # shadcn/uiコンポーネント
│   │   ├── youtube-player/   # YouTube再生コンポーネント
│   │   ├── audio-recorder/   # 録音コンポーネント
│   │   └── section-control/  # 区間設定コンポーネント
│   ├── hooks/               # カスタムフック
│   │   ├── use-youtube.ts   # YouTube API管理
│   │   ├── use-recorder.ts  # 録音機能管理
│   │   └── use-section.ts   # 区間設定管理
│   ├── lib/                 # ユーティリティ
│   │   ├── utils.ts         # 共通ユーティリティ
│   │   └── youtube.ts       # YouTube API初期化
│   ├── stores/              # Zustand ストア
│   │   ├── player-store.ts  # 再生状態管理
│   │   ├── recorder-store.ts # 録音状態管理
│   │   └── section-store.ts  # 区間設定管理
│   └── types/               # TypeScript型定義
│       └── index.ts
├── public/                  # 静的ファイル
├── docs/                    # ドキュメント
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.js
└── README.md
```

### コンポーネント設計

#### 1. YouTubePlayer コンポーネント
```typescript
interface YouTubePlayerProps {
  videoId: string;
  startTime?: number;
  endTime?: number;
  onStateChange?: (state: PlayerState) => void;
}
```

#### 2. AudioRecorder コンポーネント
```typescript
interface AudioRecorderProps {
  onRecordingComplete?: (blob: Blob) => void;
  maxDuration?: number;
}
```

#### 3. SectionControl コンポーネント
```typescript
interface SectionControlProps {
  duration: number;
  onSectionChange?: (start: number, end: number) => void;
}
```

### 状態管理設計

#### PlayerStore
```typescript
interface PlayerStore {
  videoUrl: string;
  videoId: string | null;
  playerState: PlayerState;
  currentTime: number;
  duration: number;
  setVideoUrl: (url: string) => void;
  setPlayerState: (state: PlayerState) => void;
  setCurrentTime: (time: number) => void;
}
```

#### RecorderStore
```typescript
interface RecorderStore {
  isRecording: boolean;
  recordings: Recording[];
  currentRecording: Blob | null;
  startRecording: () => void;
  stopRecording: () => void;
  saveRecording: (recording: Recording) => void;
}
```

#### SectionStore
```typescript
interface SectionStore {
  startTime: number;
  endTime: number;
  isLooping: boolean;
  loopCount: number;
  setSection: (start: number, end: number) => void;
  toggleLoop: () => void;
  setLoopCount: (count: number) => void;
}
```

## 3. 実装の推奨事項

### パフォーマンス最適化
- React.memo()を使用してコンポーネントの再レンダリングを最小化
- 音声データは IndexedDB に保存して大容量データを効率的に管理
- YouTube Player APIの遅延読み込み

### アクセシビリティ
- キーボードショートカットの実装（Space: 再生/一時停止、R: 録音開始/停止）
- ARIA属性の適切な使用
- フォーカス管理の実装

### セキュリティ
- Content Security Policy (CSP) の設定
- HTTPSでのみマイクアクセスを許可
- ユーザーデータはローカルストレージのみに保存

### 将来の拡張性
- i18n対応の準備（next-intl）
- PWA化の検討
- 音声波形表示機能の追加準備

## 4. 開発環境設定

### 必要なパッケージのインストール
```bash
# Next.js プロジェクトの作成
npx create-next-app@latest shadowing-helper --typescript --tailwind --app

# 追加パッケージのインストール
npm install zustand
npm install @types/youtube

# shadcn/ui のセットアップ
npx shadcn@latest init
npx shadcn@latest add button
npx shadcn@latest add slider
npx shadcn@latest add input
npx shadcn@latest add card
```

### 環境変数
```env
# .env.local
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## 5. デプロイ戦略

### ホスティング
- **Vercel** または **Netlify** を推奨
  - 静的サイトホスティング
  - 自動デプロイ
  - SSL証明書の自動管理

### CI/CD
- GitHub Actions を使用
- main ブランチへのプッシュで自動デプロイ
- プレビューデプロイの設定

## 6. まとめ

この技術スタックを採用することで、以下のメリットが得られます：

1. **開発効率**: Next.js + TypeScript + shadcn/ui により高速な開発が可能
2. **パフォーマンス**: 静的生成とコード分割により高速なページロード
3. **保守性**: 型安全性とコンポーネント指向により保守が容易
4. **拡張性**: モジュラーな設計により機能追加が容易
5. **コスト効率**: 静的ホスティングにより運用コストを最小化

MVP版として必要十分な機能を実装しつつ、将来的な拡張にも対応できる設計となっています。