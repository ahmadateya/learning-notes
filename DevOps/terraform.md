# Terraform Notes

* Ansible vs Terraform
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-10-01%2011-14-09.png" width="550" height="300">

* Some Terraform Commands
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-10-01%2014-34-15.png" width="550" height="300">

* you can use terraform commands `terraform state list` and `terraform state show` to get the data of your resources inside the provider like listing the EC2 instances or getting the ARN of some resource
* you can remove a resource by deleting it from the config file or using the CLI but always use the config file for the traceability 
* use `terraform plan` command its useful for showing the attributes for resources and other stuff
* use `output` keyword to have nice data after executing the configuration
* you can use variables with terraform in 3 ways: 
    1. declaring the variable inside the config file and passing its value when executing `terraform apply` will prompt you to pass the var value
    2. also declaring the variable inside the file and passing it by `terraform apply -var "var_name=value"`
    3. using a file with `.tfvars` extension writing to it key/value pairs of the variables, _the recommended way_
        * terraform.tfvars is the default name
        * or use another name and pass it while apply by --var-file
* if you set an enironment variables it will be set in the terraform app only not global in the app
* you can create your own custom env vars by using `TF_VAR` prefix
	* if you did so, you can use the var inside your config file by declaring it again without `TF_VAR` prefix 	 
