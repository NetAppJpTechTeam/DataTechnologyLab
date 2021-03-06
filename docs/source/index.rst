.. Data Technology Lab documentation master file, created by
   sphinx-quickstart on Sat Jan 13 08:55:20 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

=============================================================
NetApp Digital Transformation Lab (NDX)
=============================================================

.. image:: https://travis-ci.org/NetAppJpTechTeam/NetAppDigitalTransformationLab.svg?branch=master
    :target: https://travis-ci.org/NetAppJpTechTeam/NetAppDigitalTransformationLab

はじめに
=============================================================

目的
------------------------------------------------------------

* デジタルトランスフォーメーションを支える根幹技術とNetApp製品を組み合わせることによる価値を触りながら体験
* 特定のシナリオに沿った関連技術の組み合わせ検証
* Labを通じての技術者コミュニティの創出

連絡方法
------------------------------------------------------------

連絡先は以下を準備しています。

* 本ドキュメントのリポジトリにIssueまたはPull-request

  * GitHub リポジトリ: https://github.com/NetAppJpTechTeam/NetAppDigitalTransformationLab

* Twitter:

  * `@NetAppJPTech <https://twitter.com/NetAppJPTech>`_

* Slack:

  * `継続的なコミュニケーション用 NetAppJpTech #ndx <https://bit.ly/2HsbpHw>`_

Contents
=============================================================

基礎編
----------

コンテナ化のシナリオを実施する

.. toctree::
   :maxdepth: 2
   :caption: 基礎編: アプリケーションのコンテナ化からサービスメッシュまで

   container/index
   container/Level0/index
   container/Level1/index
   container/Level2/index
   container/Level3/index
   container/Level4/index
   container/Level5/index

-----------


.. toctree::
   :maxdepth: 2
   :caption: 実践編: 画像解析AIの作成を体験

   MLWorkflow/index
   MLWorkflow/intro/index
   MLWorkflow/eda/index
   MLWorkflow/training/index
   MLWorkflow/serve/index
   MLWorkflow/serve/pre-trained
   MLWorkflow/hybrid-deploy/index

References
=============================================================
.. toctree::
    :maxdepth: 1

    container/others/cmdreferences
    container/others/terms
    commons/references
    container/Level4/ingress/ingress


インストールするもの・あると便利なもの
=============================================================

.. toctree::
   :maxdepth: 2

   commons/installed

DesignDoc
=============================================================
.. toctree::
    :maxdepth: 2

    container/others/design

..
   やるべきこと一覧
   =============================================================

   このドキュメント全体のTODOをまとめたもの
   ..
      .. toctree::
      :maxdepth: 2

      commons/todos


   Indices and tables
   =============================================================

   * :ref:`genindex`
   * :ref:`modindex`
   * :ref:`search`
