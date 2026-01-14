# Pre Meeting
We will working on the release page [In here](https://comvex.atlassian.net/wiki/x/AgAtf)
- There was a template that name "Release Preparations"
- Page name often the release datetime. If there is a hotfix, we put the 🔥 emoij prefix.

Need to send a message to channel #eng_release_preparations [https://comvex.slack.com/archives/C03QT8UPK43] to inform who will be in charge of release:
> I'll join today's release for Corellian. We'll release:  
> - backend-app
> - web-app
> - userweb-bff
# In Meeting
The Preparation:
* meeting in Gather, the meeting often held in blue rug
* Don't need to turn on camera
* Share screen: Just let leader share the screen

Meeting Agenda:
### 1. Assign PIC of each services: 
If there are many in charge of one service. Or one person in charge in many services. We will re-assign in the document create in pre-meeting.

### 2. Run Release Please
 Go to the [release please](https://jenkins.mgmt.digima.com/job/Release%20Please/) and select services which are going to release
-> Input will be the services ready to release and blank branch

<<<<<<< HEAD
Each member will check the service their in charge and merge the release PR
=======
Each member will check the service their in charge and merge the release PRs to `develop`
>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
### 3. Create ECR
Now in the releasing project
1. fetch all change in develop and master branch
	Then `rebase develop master`
	then push the `master` branch

<<<<<<< HEAD
2. Then rebuild the Image
	* Come to jenkins and then. Ex: User-BFF image build 
		* Access: 
		* run with parameter
		* branch origin/master + uncheck cicd
		* There will be image tag have format `commit-xxxx` print in console. In this case: `commit-0f651bd`
		* We have to copy and temporary save this commit to verify the ERC tag in next step.
=======
2. Rebuild the Image with 
	* Come to jenkins and then. Ex: User-BFF image build 
		* Access: 
		* run with parameter: branch `origin/master` + uncheck cicd
		* There will be image tag have format `commit-xxxx` print in console. In this case: `commit-0f651bd`
		* We have to copy and temporary save this commit to verify the ERC tag in next step.

>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
3. Next step is promote the image. Ex: Stg user-bff promotion
	* The purpose of the promotion is let the digima-infra create a package for the release
	* Full promotion services: [link](https://jenkins.mgmt.digima.com/job/Digima%20Staging%20Userweb%20BFF%20Promotion/search/?q=promotion&Jenkins-Crumb=9bf90e73a3532bfee1493ca2ed29349f33d879a886cd88ea18931b1f8488f44f)
	* [Digima Staging Userweb BFF Promotion](https://jenkins.mgmt.digima.com/job/Digima%20Staging%20Userweb%20BFF%20Promotion/search/?q=Digima+Staging+Userweb+BFF+Promotion) 
<<<<<<< HEAD
	* Put the previous step image_tag to image_tag : `commit-0f651bd`
=======
	* Put the previous step image_tag to image_tag : `commit-0f651bd` or can use `branch-master`
>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
	* then run / check and process the promotion

4. Last step is bump the version for the next release. We need to increase the `package.yml` minor version, prepair for the next release
	* checkout branch `develop`
<<<<<<< HEAD
	* Update the package.yml
=======
	* Update the `package.yml`
>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
```
chore: Bump app version to x.x.x
```

<<<<<<< HEAD
=======
### Hotfix 
Create hotfix branch `hotfix/vx.x.1`
Then create PR to master
And then run release please with master branch target
Then just do the promotion the newly created image


>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
We have to update the print page when create release
Frontend have another way, we have to push to master for the production release.

#### Backend App
<<<<<<< HEAD
Run the promotion: https://jenkins.mgmt.digima.com/view/App%20-%20Backend/job/Digima%20Staging%20Backend%20Promotion/339/
Do we need to build????
Get the commit from the pull request which merge. For ex: https://github.com/comvex-jp/digima-backend-app/pull/5656
Then run the promotion.
=======
After release please,  

Run the promotion: https://jenkins.mgmt.digima.com/view/App%20-%20Backend/job/Digima%20Staging%20Backend%20Promotion/339/
Do we need to build???? NO

Get the commit from the pull request which merge. For ex: https://github.com/comvex-jp/digima-backend-app/pull/5656

Backend_commit: `9e8c58c15`
Infra_commit: `develop`
[ ] Rebase

![[Screenshot 2025-10-14 at 14.35.23.png]]
[`d6c3ea4`](https://github.com/comvex-jp/digima-backend-app/commit/d6c3ea4ca645dc68f7f967c61a489c245abe7a88)

Then run the promotion.

>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
For update version we have to alter the `composer.json`
https://github.com/comvex-jp/digima-backend-app/blob/develop/composer.json


# Post Meeting
