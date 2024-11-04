# JoinCounter
JoinCounterは、プレイヤーがジョインした回数をPersistenceを利用して永続保存できるシステムです。カウントの間隔はDaily、Weekly、Monthly、Yearlyから指定可能で、カウントはジョイン直後や特定エリアへの進入時など、さまざまなタイミングで行うことができます。また、回数カウントをリセットする機能も備えています。

## ファイル構成
![スクリーンショット 2024-11-03 124118](https://github.com/user-attachments/assets/57c99cb1-a32f-4eb7-9554-877fb08203a6)

- **Samples**  
  サンプルシーンや参照用のアセットが含まれるフォルダです。

- **UdonSharp**  
  UdonSharpスクリプトを配置するフォルダ。JoinCounterに必要なスクリプトが入ってます。

- **JoinCounter**  
  ジョインカウンターのメインプレハブです。プレイヤーのジョイン回数を管理し、指定されたカウント条件に基づいてカウントを増減する機能が含まれています。

- **JoinTriggerByArea**  
  特定のエリアに入った際にカウントされるプレハブです。このオブジェクト内にプレイヤーが入るとカウントされます。

- **JoinTriggerByJoin**  
  プレイヤーがワールドに入った直後にカウントされるプレハブです。

- **JoinTriggerByMethod**  
  SendCustomEventやSendCustomNetworkEventから呼び出された時にカウントされるプレハブです。他のギミックと組み合わせて使用できます。

- **ViewerObject**
  ジョイン回数を表示するビューワー用のプレハブです。このオブジェクトをPickupしたプレイヤーのジョイン回数が表示されます。表示内容は他のギミックと組み合わせて使用可能です。

## 使い方
シーン内にJoinCounterプレハブを配置します。次に、少なくとも1つ以上のJoinTrigger（例: JoinTriggerByArea、JoinTriggerByJoinなど）を配置してください。複数のJoinTriggerを組み合わせや同じJoinTriggerを複数配置することも可能です。基本的には、配置するだけでジョイン回数が保存されますが、表示処理を行うには必要に応じて各種コンポーネントの設定を行ってください。
ViewerObjectは複数配置することができます。

## 各種コンポーネントの使い方
### JoinCounter
JoinCounterプレハブにアタッチされています。ジョインカウント条件の設定や他のギミックとの連携設定が行えます。
![スクリーンショット 2024-11-03 143947](https://github.com/user-attachments/assets/e4493523-8332-49a6-bd30-705b6ccaf43e)  

#### 基本設定
| 設定項目                           | 説明                                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| **Count Type**                    | ジョイン回数をどの期間でカウントするかを指定します。「Daily」「Weekly」「Monthly」「Yearly」から選択します。「Always」は機能テスト用で毎回カウントされるようになります|
| **Boundary Time**                 | 期間の境界となる時間を設定します。 |
| **Start Day Of Week**             | Count Typeが「Weekly」の場合、週の開始曜日を指定します。|
| **Reset Type**                    | カウントリセット期間を指定します。期間条件を満たすとジョインカウントが0になります。「None」「Daily」「Weekly」「Monthly」「Yearly」から選択します。|

#### 他のギミック連携設定
`OnJoinEvent`を設定することで、ジョインカウントが行われたタイミングに他のギミックを実行することができます
| 設定項目                           | 説明                                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| **On Join Event Listeners**        | 連携するギミックのUdonを設定します。複数設定することができます。 |
| **Join Event Name**                | ジョインカウントが行われたときに実行されるイベント名をいれます。On Join Event Listenersで設定したUdon内にここで設定する名前のイベントが実装されている必要があります。 |
| **Join Event Count Variable Name** | (オプション)ジョインカウントを利用したい場合はOn Join Event Listenersで設定したUdon内に`public int`で変数を作成し、その名前を設定します。 |

`OnDataUpdate`を設定することで、ジョインカウントの数値が変更されたタイミングに他のギミックを実行することができます
| 設定項目                           | 説明                                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| **On Data Update Event Listeners**   | 連携するギミックのUdonを設定します。複数設定することができます。 |
| **Update Event Name**                | ジョインカウントの数値が変更されたときに実行されるイベント名をいれます。On Data Update Event Listenersで設定したUdon内にここで設定する名前のイベントが実装されている必要があります。 |
| **Update Event Count Variable Name** | (オプション)ジョインカウントを利用したい場合はOn Data Update Event Listenersで設定したUdon内に`public int`で変数を作成し、その名前を設定します。|
| **Update Event Last Join Time Variable Name** | (オプション)最終ジョイン時刻を利用したい場合はOn Data Update Event Listenersで設定したUdon内に`public DateTime`で変数を作成し、その名前を設定します。|

#### デバッグ機能
エディタ限定で機能します。
| 設定項目                           | 説明                                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| **Date Text**                    |指定された日付に最終ログイン時刻を設定します。入力形式は`yyyy/MM/dd HH:mm`です。 |

---

### JoinCountViewer
ViewerObjectプレハブにアタッチされています。他のギミックとの連携設定が行えます。JoinCountViewerには表示機能は入ってないため連携ギミック内に表示機能を実装する必要があります。
![スクリーンショット 2024-11-03 161740](https://github.com/user-attachments/assets/ff56d911-b1c3-4f07-81aa-0fdea7b269c7)

#### 基本設定
なし

#### 他のギミック連携設定
ViewerUdonを設定することで、ViewerObjectをPickupしたプレイヤーのジョインカウントなどを連携ギミックへ渡すことができます。
| 設定項目                           | 説明                                                                                                       |
|-----------------------------------|----------------------------------------------------------------------------------------------------------|
| **Viewer Udon**   | 連携するギミックのUdonを設定します。複数設定することができます。 |
| **View Start Event Name**                | オブジェクトをPickupしたときに実行されるイベント名をいれます。Viewer Udonで設定したUdon内にここで設定する名前のイベントが実装されている必要があります。 |
| **View End Event Name** | (オプション)オブジェクトをDropしたときに実行されるイベント名をいれます。Viewer Udonで設定したUdon内にここで設定する名前のイベントが実装されている必要があります。|
| **Player Name Variable Name** | (オプション)Pickupしたプレイヤーの名前を使用したい場合はViewer Udonで設定したUdon内に`public string`で変数を作成し、その名前を設定します。|
| **Join Count Variable Name** | (オプション)Pickupしたプレイヤーのジョイン回数を使用したい場合はViewer Udonで設定したUdon内に`public int`で変数を作成し、その名前を設定します。|
| **Last Join Time Variable Name** | (オプション)Pickupしたプレイヤーの最終ジョイン時刻を使用したい場合はViewer Udonで設定したUdon内に`public DateTime`で変数を作成し、その名前を設定します。|

---

### JoinTriggerByArea 
JoinTriggerByAreaプレハブにアタッチされています。設定項目はありません。BoxColliderなどのColliderと併用し、Colliderは`IsTrigger`にチェックを入れてください。  
![スクリーンショット 2024-11-03 144008](https://github.com/user-attachments/assets/738b2465-3810-49c2-bbb0-67464c588f2b)  

---

### JoinTriggerByJoin
JoinTriggerByJoinプレハブにアタッチされています。設定項目はありません。
![スクリーンショット 2024-11-03 144019](https://github.com/user-attachments/assets/e6c3255e-8514-40c0-8cd1-54a6a76a669c)  

---

### JoinTriggerByMethod
JoinTriggerByMethodプレハブにアタッチされています。設定項目はありません。
Joinイベントが実装されているため、`SendCustomEvent("Join");`などを使用して動作させます。
`SendCustomNetworkEvent`を使用する場合は、`Synchronization Method`をManualまたはContinuousに設定して、同期可能な状態にしてください。

![スクリーンショット 2024-11-03 144031](https://github.com/user-attachments/assets/168f0f44-fe5f-4ecb-8837-f65f2b5edb1d)  


## サンプルシーンについて
サンプルシーンには、各種連携ギミックが含まれており、ジョイン回数の増加やカウントアップアニメーション、ビューワー処理などが実装されています。プロジェクトに応じて設定を参考にしてください。
