# Final steps

Assuming everything has been applied correctly you should find the URL the service is hosted at e.g. https://workshop-run-lfdmquzetq-lz.a.run.app at the bottom of the last apply output
Copy your link and paste it in a browser and add **/hello** at the end (e.g. **https://workshop-run-lfdmquzetq-lz.a.run.app/hello**), this should return a page with the text **I'm alive redis at xxx.xxx.xxx.xxx:6379** and shows that the service is running and has gotten the secrets.

To actually add somthing to the redis instance you can navigate to the Cloud Run url. It should show a page with **Key values in redis** at the top and a basic form where you can enter a key value pair.
When you press submit you should get a **Success** page which means it has stored it to redis, if you now reload the root url it should show your new key/value pair that was fetched from redis

You can also list all the key/value pairs in the workshop redis db paste the Cloud Run url and add **/list** (e.g. **https://workshop-run-lfdmquzetq-lz.a.run.app/list**)

## Production env

Now we can create our production environment by simply making a **prod.tfvars** and creating a new workspace.
Put something like this into your **prod.tfvars**
```
project_id = "yourname-terraform-workshop-prod"
project_region = "europe-north1"
environment = "prod"
```

Then we run terraform and once it is complete we should have a fully ready prodiction environment that is configured exactly like our dev environment
```
terraform workspace new prod
terraform plan -var-file prod.tfvars -out prod.plan
terraform apply prod plan
```

If the cloud access is properly configured to not give anyone write/admin rights you can always be sure that the environment is exactly like defined in the terraform code.

Now we should clean up after ourselves so we don't waste resources.
```
terraform workspace select prod
terraform destroy -var-file prod.tfvars 
terraform workspace select dev
terraform destroy -var-file dev.tfvars
```

[previous](https://github.com/rselbo/TerraformWorkshop/blob/main/04-CloudRunAndSecrets.md)