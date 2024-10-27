##This project tried parameters with main.tf without using a Jenkinsfile. 
I used the choice parameter. To destroy the resources we created, I used a boolean parameter. I wrote code in the shell execution so that if this parameter is selected, it will not apply.

###The command to be written in the execute shell is as follows:


WORKSPACE_NAME=$(basename "$WORKSPACE")
echo "Seçilen çalışma alanı: ${WORKSPACE_NAME}"
terraform workspace select "${WORKSPACE_NAME}" || terraform workspace new "${WORKSPACE_NAME}"

KEY_NAME="${WORKSPACE_NAME}-key"
if ! aws ec2 describe-key-pairs --key-name "$KEY_NAME" --region us-east-1 >/dev/null 2>&1; then
    echo "Key Pair '$KEY_NAME' mevcut değil, oluşturuluyor..."
    aws ec2 create-key-pair --key-name "$KEY_NAME" --query 'KeyMaterial' --output text --region us-east-1 > "$KEY_NAME.pem"
    chmod 400 "$KEY_NAME.pem"
else
    echo "Key Pair '$KEY_NAME' zaten mevcut, işlem atlanıyor."
fi


terraform init


if [ "${DESTROY_RESOURCES}" == "true" ]; then
    echo "Kaynaklar yok ediliyor..."
    terraform destroy --auto-approve -var="ins_type=${instance_type}" -var="ins_ami=${AMI}" -var="keypair=${KEY_NAME}"
else
    echo "Kaynaklar oluşturuluyor..."
    terraform apply --auto-approve -var="ins_type=${instance_type}" -var="ins_ami=${AMI}" -var="keypair=${KEY_NAME}"
fi
