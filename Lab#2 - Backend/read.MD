# Konfigurace Remote Backend pomocí S3 a DynamoDB

## Konfigurace S3 Bucketu
- **force_destroy**: bucket bude pomocí `terraform destroy` odstraněn i když obsahuje TF State soubor
- **aws_s3_bucket_versioning**: lze zobrazit předchozí verze TF State souboru v bucketu
- **aws_s3_bucket_server_side_encryption_configuration**: obsah bucketu je chráněn šifrováním

## Předpoklady
1. Nainstalovaný Terraform a AWS CLI

## Lab

1. Vytvořte IAM Uživatele s programatickým přístupem pomocí IAM Role

2. Nakonfigurujte AWS CLI:
    ```bash
    aws configure --profile=Backend-profile
    ```

3. Vytvořte S3 bucket a DynamoDB nejprve lokálně (TF State nelze ukládat do bucketu, který neexistuje):
    ```bash
    terraform init # Vytvoření lokálního TF State souboru
    terraform plan
    terraform apply
    ```

4. V AWS konzoli zkontrolujte:
    - prázdný DynamoDB
    - prázdný S3 bucket

5. Přesuňte lokální TF State do remote Backend:
    1. Nahraďte jméno bucketu a DynamoDB table za output v terminálu.
    2. Odkomentujte /* */ v `main.tf`:
        ```hcl
        backend "s3" {
            bucket = "<jméno vašeho bucketu>"
            key = "State/terraform.tfstate"
            region= "us-east-2"
            dynamodb_table = "<jméno vašeho DynamoDB table>"
            encrypt        = true
            profile = "Backend-profile"
        }
        ```

6. Inicializujte remote Backend:
    ```bash
    terraform init # Vytvoření remote Backend
    ```

7. V AWS konzoli zkontrolujte:
    - `terraform.tfstate` uvnitř S3 bucketu
    - (LockId uvnitř DynamoDB)

8. Vyzkoušejte použít příkaz pro manuální odemknutí (`<LOCK_ID>` lze zístat z error zprávy v terminálu):
    ```bash
    terraform force-unlock <LOCK_ID>
    ```

9. Odstraňte infrastrukturu:
    ```bash
    terraform destroy
    ```
