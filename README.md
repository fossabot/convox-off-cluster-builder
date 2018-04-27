# convox-off-build-off-cluster
Tool to create Convox Builds off the Convox cluster.

# Usage
The new `convox-build-off-cluster` tool has been overhauled and its functionality has been expanded.
For the new and improved version of the `convox-build-off-cluster` tool an adjusted version of the `.drone.yml` file is needed

# Configuration Example (_.drone.yml_)
```
convox_ninja:
  image: getterminus/cci-build-golang:20180426
  environment:
	- AWS_REGION=us-east-1
  secrets:
	- source: ninja_convox_host
	  target: convox_host
	- source: ninja_convox_password
	  target: convox_password
	- source: ninja_aws_account
	  target: aws_account
	- source: ninja_aws_access_key_id
	  target: aws_access_key_id
	- source: ninja_aws_secret_access_key
	  target: aws_secret_access_key
	- source: ninja_repo
	  target: repo
  commands:
	- convox-build-off-cluster -app=<your app name> -description=${DRONE_COMMIT_BRANCH} -gitsha=${DRONE_COMMIT_SHA} -repo $${REPO}
	- mkdir scratch
	- mv ./docker-compose.convox.yml scratch/docker-compose.yml
	- cd ./scratch
	- convox build --app=<your app name>
```

## Define these "Secrets" from the drone UI (Hamburger Menu -> Secrets)
**These settings will need to be provided by a member of the SRE team**
  * ninja_aws_account
  * ninja_aws_access_key_id
  * ninja_aws_secret_access_key
  * ninja_repo

## Additionally, set the AWS_REGION
```yaml
environment:
  - AWS_REGION=us-east-1
```
Adjust for your region (as of now it's only 'us-east-1') as needed
## Last, but not least, there is one more thing to take care of
### This step is a cane, but it is needed to get the RELEASEID.
This step prevents convox from actually building the service yet again.
```yaml
commands:
  - mkdir scratch
  - mv ./docker-compose.convox.yml scratch/docker-compose.yml
  - cd ./scratch
  - convox build --app=<your app name>
```
The `convox build...` command picks up the image that has been built and pushed by the previous `convox-build-off-cluster` tool and turns it into a release.

# Notes for the members of the SRE team:
Find all these settings in `1Password`
* **ninja_aws_access_key_id**
  * Find `ninja_dronebuilduser` in `1Password`
  * Copy **aws_access_key_id**'s value into **ninja_aws_access_key_id** in drone's Secrets UI
* **ninja_aws_secret_access_key**
  * Find `ninja_dronebuilduser` in `1Password`
  * Copy **aws_secret_access_key_id**'s value into **ninja_aws_secret_access_key** in drone's Secrets UI
* **ninja_repo**
  * Log into AWS Ninja
  * Select `Elastic Container Service`
  * Select `Repositories`
  * Filter for `offclusterbuild`
  * Click on `offclusterbuild`
  * Click on `View Push Commands`
  * Copy a string like this from step #5: `119585928394.dkr.ecr.us-east-1.amazonaws.com/offclusterbuild`
  * Paste it into **ninja_repo**'s value in drone's Secrets UI
* **ninja_aws_account**
  * Same steps as for the **ninja_repo** to get to this needed information
  * Copy the first section of the **ninja_repo** string that's all numbers; that's the AWS account number
  * Paste it into **ninja_aws_account**
