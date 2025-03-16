```mermaid
sequenceDiagram
    participant User
    participant WebApp
    participant AuthServer
    participant API
    participant Database

    User->>WebApp: ログインリクエスト
    WebApp->>AuthServer: 認証処理
    alt 認証成功
        AuthServer-->>WebApp: 認証成功
        WebApp->>User: ログイン成功メッセージ
        
        par APIリクエスト開始
            WebApp->>API: データ取得リクエスト
            API->>Database: DBアクセス
            Database-->>API: データ取得完了
            API-->>WebApp: データ送信
        and ローカルキャッシュチェック
            WebApp->>WebApp: キャッシュデータ確認
        end

        alt データ取得成功
            WebApp->>User: データ表示
        else データ取得失敗
            loop 3回リトライ
                WebApp->>API: 再リクエスト
                API-->>WebApp: データ送信
            end
            alt まだ失敗
                WebApp->>User: データ取得エラー
            else 成功
                WebApp->>User: データ表示
            end
        end

    else 認証失敗
        AuthServer-->>WebApp: 認証失敗
        WebApp->>User: ログインエラーメッセージ
    end
