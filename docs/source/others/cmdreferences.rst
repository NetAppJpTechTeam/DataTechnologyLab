=============================================================
コマンドリファレンス
=============================================================


kubectlの使い方・本家へのリンク
==============================================================

公式ガイドへのリンクです。
詳細や使い方等については以下ページを見ることをおすすめします。
このページではよく使うコマンドについてユースケースでまとめました。

* https://kubernetes.io/docs/reference/kubectl/overview/
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/
* https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/
* https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/



デプロイメントの実施
--------------------------------------------------------------

kubectl create/apply/patch/replaceを使用します。

それぞれ便利な点・留意する点があります。

* https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/#imperative-object-configuration

kubectl create デプロイメントの基本系、マニフェストファイルを指定してデプロイし、新規に行う場合に使用します。

.. code-block:: console

    $ kubectl create -f deployment.yaml


kubectl applyはcreate/replaceを包含できます。差分反映のアルゴリズムを理解して利用しましょう。
applyの動きとしてはすでにデプロイされていれば更新を行い、デプロイされていなければ新規作成の動きをします。

.. code-block:: console

    $ kubectl apply -f deployment.yaml


kubectl replace は稼働中のアプリケーションに対して動的に定義を反映する。

.. code-block:: console

    $ kubectl apply -f deployment.yaml


kubectl patch は稼働中のアプリケーションに対して、一部のフィールドを書き換える用途に使用。


状況確認
--------------------------------------------------------------

基本形としては  ``kubectl get オブジェクト名`` と ``kubectl describe オブジェクト名`` になります。
以下は ``kubectl get`` ですが、``get`` を ``describe`` に変更することで詳細な情報が確認できるようになります。

よく使うものとしては以下の通りです。

.. code-block:: console

    $ kubectl get pod

    NAME                               READY     STATUS    RESTARTS   AGE
    wordpress-mysql-58cf8dc9f9-t2wnr   1/1       Running   0          2d

.. code-block:: console

    $ kubectl get svc

    NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
    kubernetes        ClusterIP   10.96.0.1    <none>        443/TCP    10d
    wordpress-mysql   ClusterIP   None         <none>        3306/TCP   2d


.. code-block:: console

    $ kubectl get deployment

    NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    wordpress-mysql   1         1         1            1           2d


ネームスペースを指定する場合は ``-n`` オプション、または ``--all-namespaces`` で全ネームスペースのオブジェクトを確認できます。

.. code-block:: console

    $ kubectl get all -n ネームスペース名


マニフェストファイルを使用している場合は ``get`` の引数に ``-f マニフェストファイル`` を指定すると関連するオブジェクトをすべて表示してくれます。

.. code-block:: console

    $ kubectl get -f deployment.yaml


現状のオブジェクトをすべて確認する場合はオブジェクトを指定する箇所に ``all`` を設定するとすべてのオブジェクトを確認できます。


.. code-block:: console

    $ kubectl get all [-n ネームスペース名]


すべてのネームスペースのすべてのオブジェクトを確認したい場合は以下のとおりです。

.. code-block:: console

    $ kubectl get all --all-namespaces


マニフェストファイルを使用したオブジェクトの確認もできます。

``-f`` オプションを使用してデプロイ時に使用したマニフェストファイルを指定すると関連するオブジェクトをすべて表示します。

.. code-block:: console

    $ kubectl get -f wordpress-mysql-deploy.yaml
    NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
    svc/wordpress-mysql   ClusterIP   None         <none>        3306/TCP   2d

    NAME                 STATUS    VOLUME                         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pvc/mysql-pv-claim   Bound     default-mysql-pv-claim-b5e95   20Gi       RWO            ontap-gold     2d

    NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deploy/wordpress-mysql   1         1         1            1           2d


問題の特定方法について
--------------------------------------------------------------

マニフェストを

``kubectl get`` と ``kubectl describe``, ``kubectl logs`` を組み合わせて問題箇所を特定していきます。


よく使うコマンド
^^^^^^^^^^^^^^^^^

* kubectl describe オブジェクト名
* kubectl describe -f deployment.yaml

トラブルシュートの流れ
^^^^^^^^^^^^^^^^^

#. 問題箇所の特定

    #. ``kubectl get -f deployment.yaml`` で予期しない動作をしている箇所を発見
    #. kubectl describe -f deployment.yaml

#. うまく行っていない箇所が分かれば該当のPodを確認する

    #. kubectl logs pod ポッド名
    #. 3rd party製の ``stern`` というツールもあります。こちらは複数のPodに対して ``kubectl logs`` を実行する動きをします。非常に便利なものになります。

#. 取得できた情報を元に対応実施

    #. マニフェストファイルの修正

