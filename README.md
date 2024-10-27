This project tried parameters with main.tf without using a Jenkinsfile. 

The command to be written in the execute shell is as follows:


WORKSPACE_NAME=$(basename "$WORKSPACE")
KEY_NAME="${WORKSPACE_NAME}-key"
aws ec2 describe-key-pairs --key-name "$KEY_NAME" --region us-east-1 >/dev/null 2>&1
if [ $? -ne 0 ]; then
    echo "Key Pair '$KEY_NAME' mevcut değil, oluşturuluyor..."
    aws ec2 create-key-pair --key-name "$KEY_NAME" --query 'KeyMaterial' --output text --region us-east-1 > "$KEY_NAME.pem"
    chmod 400 "$KEY_NAME.pem"
else
    echo "Key Pair '$KEY_NAME' zaten mevcut, işlem atlanıyor."
fi
terraform workspace select ${WORKSPACE_NAME} || terraform workspace new ${WORKSPACE_NAME}
terraform init
terraform apply --auto-approve -var="ins_type=${instance_type}" -var="ins_ami=${AMI}" -var="keypair=${Keypair}"
