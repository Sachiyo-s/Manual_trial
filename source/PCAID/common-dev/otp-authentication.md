---
sidebar_position: 15
---

# 15. OTP認証

## 前提条件

- HTTPリクエストで使用し、`User-Agent` ヘッダーを利用する
- 作成側と検証側の両者で、共通の `salt` とする 16 バイト以上のランダム文字列を共有する

:::info `salt` について

この仕様における salt とは、パスワードをハッシュ化して保存する際に利用する salt とは意味合いが異なる点に注意してください。  
OTP認証においては、ワンタイムパスワードを生成するために、共通シークレットを特定の手順で作成します。  
この仕様における salt は、上記のシークレット作成手順において使用する、利用サービスごとのシード値として定義しています。

OTP認証を利用するサービスにおいては、salt をデータベースや環境変数などで保持することを想定しています。

:::

## OTP認証手順

- OTP認証には `Authorization` ヘッダーを使用する
- ヘッダー値のプレフィックスとする認証タイプは `Totp` （Time-based One-Time Password）を指定する

```http
Authorization: Totp eac38d043df30f177d7bb1ee2c7f20a039c5959637810cc48ba02686dd1f0bbd
```

## OTP作成方法

1. `User-Agent` + `salt` の連結文字列から、UTF-8 のバイト配列に変換したものを `secret` とする
2. UNIX日時を分単位に平準化したものを `timeStep` とする
    - 1:00:00 と比較して、1:00:59 は同じ値になり、1:01:00 は異なる値になる
3. `secret` を共通鍵としたHMAC-SHA256 を使用して、`timeStep` のバイト配列についてハッシュ計算する
4. ハッシュ値であるバイト配列をBase64URL の文字列に変換する

```cs
/// <summary>
/// ワンタイムパスワード（OTP）を作成します｡
/// OTPは、UNIX日時を分単位のステップとして扱い、URLセーフな文字列のみで生成します。
/// </summary>
/// <param name="userAgent">User-Agentヘッダー値（OTP生成のSecret要素）</param>
/// <param name="salt">ソルト値（OTP生成のSecret要素）</param>
/// <param name="unixTimestamp">UNIX日時</param>
/// <param name="stepSkew">現在時刻を基準として指定ステップだけ調整する（デフォルト0で調整なし）</param>
/// <returns>URLセーフな文字列で生成したワンタイムパスワード（OTP）</returns>
string CreateOtpCore(string userAgent, string salt, long unixTimestamp, int stepSkew = 0)
{
    if (string.IsNullOrEmpty(userAgent))
    {
        throw new ArgumentNullException(nameof(userAgent));
    }

    if (string.IsNullOrEmpty(salt))
    {
        throw new ArgumentNullException(nameof(salt));
    }

    const int rangeMinutes = 60 * 1; // 1分間（60秒）をステップ単位とする

    var secret = Encoding.UTF8.GetBytes($"{userAgent}_{salt}");
    var timeStep = (unixTimestamp / rangeMinutes) + stepSkew;

    using var hmac = new HMACSHA256(secret);

    return Base64UrlEncoder.Encode(hmac.ComputeHash(BitConverter.GetBytes(timeStep)));
}
```

## OTP検証方法

1. 基本的に、上記のOTP作成方法により、自身で作成したOTPと渡されたOTPが一致すれば認証OKとする
2. 多少の時刻ズレを考慮して、`timeStep` を過去や未来に調整したOTPも比較対象とする
3. `salt` のローテーションを考慮して、検証側は複数の `salt` を受け付けるようにして、それぞれのOTPを比較対象とする

```cs
/// <summary>
/// ワンタイムパスワード（OTP）を検証します｡
/// 未来方向と過去方向のそれぞれに、指定ステップ（分単位）だけ許容値を設定できます。
/// 前後1ステップ以上を許容する（現在､過去､未来のステップを検証対象とする）ことを想定しています｡
/// </summary>
/// <param name="verifyOtp">検証対象のワンタイムパスワード（OTP）</param>
/// <param name="userAgent">User-Agentヘッダー値（OTP生成のSecret要素）</param>
/// <param name="salts">有効なソルト値の一覧（OTP生成のSecret要素）</param>
/// <param name="pastStepSkew">過去方向のステップ許容値（正数のみ）</param>
/// <param name="futureStepSkew">未来方向のステップ許容値（正数のみ）</param>
/// <returns>正常なワンタイムパスワード（OTP）ならtrue、それ以外はfalse</returns>
bool VerifyOtp(string verifyOtp, string userAgent, string[] salts, int pastStepSkew, int futureStepSkew)
{
    if (salts == null || salts.Length == 0 || salts.Any(x => string.IsNullOrEmpty(x)))
    {
        throw new ArgumentNullException(nameof(salts));
    }

    if (pastStepSkew < 0 || futureStepSkew < 0)
    {
        throw new InvalidOperationException("ステップの許容値に負数は指定出来ません。");
    }

    var timestamp = GetCurrentTimestamp();

    const int currentStep = 0;
    var allowPastSkews = Enumerable.Range(1, pastStepSkew).Select(s => -1 * s);
    var allowFutureSkews = Enumerable.Range(1, futureStepSkew).Select(s => s);
    var allowStepSkews = new[] { currentStep }.Concat(allowPastSkews).Concat(allowFutureSkews);

    // 有効なSaltごとのステップ許容範囲において、検証対象のOTPと一致判定して、一致が見つかればOKとする
    return salts.Any(salt =>
    {
        return allowStepSkews.Any(skew =>
        {
            return verifyOtp == CreateOtpCore(userAgent, salt, timestamp, skew);
        });
    });
}
```

### 実装サンプル

[OTPサンプル.zip](./materials/OTPサンプル.zip)
