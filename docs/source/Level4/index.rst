=============================================================
Level 4: 運用編
=============================================================

目的・ゴール: ラボを実施する環境の確認
=============================================================

本番運用になった場合に必要となるアプリケーションの可用性、インフラの可用性、非機能要件の実現について検討します。

Level3まででアプリケーションを迅速にリリースする仕組みを作成しました。ここからはマスタのHA化やバックアップをどうするかを検討します。


流れ
=============================================================


本レベルでは運用時に課題となり、解決が必要となる項目についてリストしました。
現在の環境に対して変更をして見てください位。ここでは実際に手をうごかしてもらってもいいですし、
チーム内でディスカッションの材料にしてください。


アプリケーションの可用性を向上させる
=============================================================

アプリケーションの可用性を挙げるためWorkload APIを使用する
-------------------------------------------------------------

すでに ``Deployment`` で使われているかもしれませんが、replica数などを定義できます。一般的なアプリケーションデプロイに使用します。

各ノードでコンテナを稼働させる ``DaemonSet`` があります。ログ収集用のデーモンやメトリクス収集などのユースケースがあります。

レプリカ生成時の順序制御、各ポッドにPVを割り当てることができる ``StatefulSet`` があります。主にクラスタ、分散環境におけるユースケースで使用するものです。

kubernetes上のオブジェクト名は以下の通りです。

* ReplicaSet
* DaemonSet
* StatefulSet


ローリングアップデート
-------------------------------------------------------------



``Deployment`` の ``Pod template``部分に変更があった場合に自動でアップデートできます。

.. code-block:: console

    $ kubectl set image deployment/DEPLOYMENT CONTAINER=IMAGE_NAME:TAG

``--record``オプションをつけるとアノテーションが付与されます。

参考: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


リリース後に問題発生、アプリケーションを戻す
-------------------------------------------------------------

アプリケーションは ``Deployment`` でリビジョン管理しています。

``rollout history`` でリビジョンを確認できます。


.. code-block:: console

    $ kubectl rollout history deployment/デプロイメント名

    deployments "nginx-deployment"
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>


各リビジョンの詳細については ``--revision=N`` を付与することで詳細を確認できます。

.. code-block:: console

    $ kubectl rollout history deployment/nginx-deployment --revision=2
    deployments "nginx-deployment" with revision #2
    Pod Template:
      Labels:       app=nginx
            pod-template-hash=1520898311
      Containers:
       nginx:
        Image:      nginx:1.9.1
        Port:       80/TCP
        Environment:        <none>
        Mounts:     <none>
      Volumes:      <none>

アプリケーションは ``Deployment`` でリビジョン管理しており、ロールバック機能も提供しています。
``rollout undo`` で直前のリビジョンに戻ります。``--to-revision``を指定することで任意のリビジョンに戻すことも可能です。


.. code-block:: console

    $ kubectl rollout undo deployment/nginx-deployment [--to-revision=N]

保存されるリビジョンは ``revisionHistoryLimit`` で定義できるため、運用に合わせた数にしましょう。

Helmを使った場合にも同様のことが実現可能です。

アプリケーション負荷に応じたスケールアウト・イン
-------------------------------------------------------------

``Horizontal Pod Autoscaler`` を使用してアプリケーションの負荷に応じてスケールアウトすることができます。

事前定義としてアプリケーションの負荷情報をheapsterで収集しておく必要があります。
以下の例はすべてのポッドのCPU使用率の平均が50％を超えた場合にレプリカを最大10まで増やす動作をします。

.. code-block:: console

    $ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10


上記の例では、CPU使用率をメトリクスとしていますが複数のメトリクスをしようしたり、カスタマイズすることも可能です。

参考: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

インフラの可用性を向上させる
=============================================================


k8s Master の冗長化
-------------------------------------------------------------

API受付をするマスタ系のノードやetcdやkubernetesサービスの高可用性も担保しましょう。

また、障害設計をどの単位(DC単位、リージョン単位）で行うかも検討をしましょう。

* https://kubernetes.io/docs/tasks/administer-cluster/highly-available-master/
* https://github.com/kubernetes/community/blob/master/contributors/design-proposals/cluster-lifecycle/ha_master.md


ログの確認、デバッグ方法
-------------------------------------------------------------

標準のkubectlだとログがおいづらいときがあるため以下のツールの検討をします。

* kubernetesホストの/var/log/containerにログは保管。(systemd系の場合）
* sternなどのログ管理ツールを活用する
* fluetdを使いログを集約する。

コンテナクラスタの監視
-------------------------------------------------------------

監視する対象として、メトリクス監視、サービス呼び出し、サービスメッシュなど分散環境になったことで従来型のアーキテクチャとは違った監視をする必要があります。
簡単にスケールアウトできる＝監視対象が動的というような考え方をします。

また、分散環境では１つのアプリケーションでも複数のサービス呼び出しがおこなわれるため、どのようなサービス呼び出しが行われているかも確認できる必要があります。

* (heapster|Prometheus) + Grafana + InfluxDB を使いコンテナクラスタの監視。
* 分散環境に於けるサービスの呼び出しを可視化 Traces = Zipkin
* ServiceGraph Graphbiz & Prometeus
* ServiceMesh

バックアップはどうするか？
-------------------------------------------------------------

大きく以下のバックアップ対象が存在。

* etcd のバックアップ戦略
* コンテナ化されたアプリケーションの永続化データ
* コンテナイメージ

セキュリティアップグレード
-------------------------------------------------------------

例えば、脆弱性があった場合の対処方法はどうすればよいか。

* ノードごとにバージョンアップするため、ある程度の余力を見込んだ設計とする。
* kubectl drain を使いノードで動いているPodを別ノードで起動、対象ノードをアップデートし、ポッドを戻す。

DRをどうするか？
-------------------------------------------------------------

アプリケーションのポータビリティはコンテナで実現。
別クラスタで作成されたPVはそのままは参照できないので以下の方法を検討する。

* Cluster federation
* CSI (Container Storage Interface)の既存ボリュームのインポートに対応をまつ
* Heptio ark: https://github.com/heptio/ark + SnapMirror



