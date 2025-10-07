## Preparation 
release tags, after deployment we have task in Release tasks: the task that need to be done in realease step,

# Pre-release
- Update version.yaml in project
- Run the release please in jenkin [here](https://jenkins.mgmt.digima.com/job/Release%20Please/)
# Release 

But basically, the ComvexEngineering yeah this is a service account. Then when you run release-please in jenkins, this will create a release PR for the repository, then after merging and confirm, the Release Please jenkins job will publish a release in Github


Should create topic first?
Should return 

---
## Process
After the [[2. Areas/comvex/meetings/Release Prepairation/README|Release Preparation]]
Now we already have the release tag in repository. Verify by check the digma-infra tags [here](https://github.com/comvex-jp/digima-infra/tags)
The Deplopment we only need to run the production release
Jenkins parameter:
- infra_branch: `v3.19.0-sms-api` Have to check is it exist in `digima-infra`
- image_tag: `v3.19.0`
