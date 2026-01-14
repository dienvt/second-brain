
## 🪜Prepare release
https://88-oct.atlassian.net/wiki/spaces/OCT/pages/3232727254/Howto+Submit+weekly+release


Sample tag version:

Sample Release Note: [https://88-oct.atlassian.net/wiki/spaces/OCT/pages/3409052005/20250304_RemoteCall_Release](https://88-oct.atlassian.net/wiki/spaces/OCT/pages/3409052005/20250304_RemoteCall_Release)

create release tag

|BE||
|---|---|
|Web||
|Mobile||

Submit Weekly Release: [https://88-oct.atlassian.net/wiki/x/1oCvw](https://88-oct.atlassian.net/wiki/x/1oCvw)

Week Release Sample

```
リリースノートURL
<https://88-oct.atlassian.net/wiki/spaces/OCT/pages/3242590467/20240924_RemoteCall_Release>
リリース担当者
Khanh Nguyen, Tuna
リリース日時
2025/03/18 16:00:00+07:00
リリース作業開始日時
2025/03/18 16:00:00+07:00
計画的 or 臨時？
計画的リリース
リリース予定のリポジトリ
<https://github.com/88labs/andpad-vanguard-mobile/tree/main/flutter/apps/remote-instruction>
<https://github.com/88labs/andpad-vanguard-web/tree/main/apps/remote>
<https://github.com/88labs/andpad-vanguard-backend/tree/main/go/services/remote>
リリースバージョン
#remote_v1.40.0 #20250422_release #20250422_remote_release #20250422_remote-bff_release
依頼者

@Tuna Nguyen
補足
Weekly release for the Remote Call Project.
リモート通話の定期リリースです。

```


## Reference
https://andpadvietnam.slack.com/archives/C04L64WH39P/p1717394636308339

### Material
#### Step one: create a release tag
Access: [git release](https://github.com/88labs/andpad-vanguard-backend/releases)
* Hit "Draft a new release"
* Create new tag with format YYYYMMDD_remote_release
	* YYYYMMDD: the day we planned to release
	* remote: prj name, using `remote` for both `remote-bff` and `remote`
* Select previous tag: The latest release tag before the planning tag
* Hit generate release notes
* Edit the description of the release.
* Tick "Set as a pre-release"
* Then publish the release tag

Open release thread then announce about the release tag, sample:

=======
BE Release  

- Release tag: [https://github.com/88labs/andpad-vanguard-backend/releases/tag/20240604_remote_release_2nd](https://github.com/88labs/andpad-vanguard-backend/releases/tag/20240604_remote_release_2nd)

* Release service:
	* Remote Service
	* Remote BFF Service

- Run patch: [https://github.com/88labs/andpad-vanguard-backend/tree/patch/production-remote-role-permission-20240604](https://github.com/88labs/andpad-vanguard-backend/tree/patch/production-remote-role-permission-20240604)
=======

#### Step two: manual deploy

Access [github action](https://github.com/88labs/andpad-vanguard-backend/actions)
Choose: [manual-deploy](https://github.com/88labs/andpad-vanguard-backend/actions/workflows/manual-deploy.yml)
Hit run, choose tag and project name
Capture screen then send to Release thread.
Hit "Run workflow"
Also send link deployment into slack
Testing mobile app (Anh Tai)
![[Screenshot 2024-04-08 at 11.18.24.png]]





## Planning release

#### 20240723
Tag: [20240723_remote_release](https://github.com/88labs/andpad-vanguard-backend/releases/tag/20240723_remote_release)
