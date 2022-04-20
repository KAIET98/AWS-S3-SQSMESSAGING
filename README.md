# AWS-S3-SQSMESSAGING
El objetivo de este repositorio es crear una conexión entre S3 y SQS, donde lleguen mensajes a SQS diciendo que se han subido objetos al S3

## 1. Creación de un queue en SQS. 

1. Primero de todo, vamos a SQS y creamos un queue. En el tipo de SQS, en este tutorial seleccionaremos el "Standard" y como nombre le daremos:

```
EventFromS3
```
2. Configuración del queue. Vamos a dejarlo tal cual para hacerlo sencillo en este tutorial. 

4. Access Policies: 

Es decir quién puede acceder al queu, por defecto solo el propietario del queue puede mandar mensajes. Pero esto se puede cambiar dando permisos. Esto lo puedes editar por medio del Basic o el Advanced. Nosotros lo haremos con el Basic. 

Entonces, en el emisor lo dejamos como "Only the queue owner". 

Y en el receptor tambien lo dejamos ocmo "Only the queue owner". 

5. Encryption: 

Vamos a habilitar el "Server-side encryption" y en el Key Type vamos a darle "AWS Key Management Services Key (SSE-KMS)". 
Dejaremos el Customer master key con el default que nos viene  y lod emas lo dejamos por defecto. 

6. Creamos el queue.

7. Entramos al queue y en "Access policies" vamos a tener que mofidicarlo para permitir que el S3 escriba al SQS. 

Le damos a info, vamos a la documentacion, Google: s3 event into SQS access policy y nos hacemos con este policy:

```
{
    "Version": "2012-10-17",
    "Id": "example-ID",
    "Statement": [
        {
            "Sid": "example-statement-ID",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SQS:SendMessage"
            ],
            "Resource": "arn:aws:sqs:Region:account-id:queue-name",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:s3:*:*:awsexamplebucket1"
                },
                "StringEquals": {
                    "aws:SourceAccount": "bucket-owner-account-id"
                }
            }
        }
    ]
}



```

En si necesitamos por un lado, permitir que el bucket escriab en el SQS. Por lo que como aun no hemos creado un bucket ni nada. Vamos a dejar las configuraciones
de permisos by default que nos da AWS. Aunque de un error. Y cuando creamos el Bucket volveremos a  editar esto.



## 2. Creamos el S3 Bucket

1. Create Bucket

Nombre: 
```
demo-sqs-queue-access-policy-tutorial
```
2. Entramos dentro del bucket>properties

En "Event Notifications" le damos a "Create Event Notifications"

Nombre: 
```
NewObjects

```
Lo vamos a crear para todos los prefijos y sufijos. 

3. Event types: seleccionamos "All object create events". 

4. Destination: seleccionamos "SQS", sleecinamos "Choose from your SQS queues", y seleccioamos el que cabamos de crear.


```
EventFromS3
```

Y le damos a "Save Changes". 

Nos a un error. 

## 3. Modificación del SQS. 

Vamos al SQS que acabamos de crear y en AccessPolicy, nos encontramos con esto: 

```
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__owner_statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::070307590085:root"
      },
      "Action": "SQS:*",
      "Resource": "arn:aws:sqs:us-east-1:070307590085:EventFromS3"
    }
  ]
}
```
Nosotros tenemos que darle permiso a S3 para que escriba en SQS, por lo que pillamos la siguiente plantilla
```
{
    "Version": "2012-10-17",
    "Id": "example-ID",
    "Statement": [
        {
            "Sid": "example-statement-ID",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SQS:SendMessage"
            ],
            "Resource": "arn:aws:sqs:Region:account-id:queue-name",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:s3:*:*:awsexamplebucket1"
                },
                "StringEquals": {
                    "aws:SourceAccount": "bucket-owner-account-id"
                }
            }
        }
    ]
}
```

Y rellenamos, dando acceso, con el recurso de s3, con su nombre de bucket y con nuestro id un numero de 12 digitios: XXXX-XXXX-XXXX. Quedaria de este estilo

```

{
    "Version": "2012-10-17",
    "Id": "example-ID",
    "Statement": [
        {
            "Sid": "example-statement-ID",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "SQS:SendMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:070307590085:EventFromS3",
            "Condition": {
                "ArnLike": {
                    "aws:SourceArn": "arn:aws:s3:*:*:demo-sqs-queue-access-policy-tutorial"
                },
                "StringEquals": {
                    "aws:SourceAccount": "XXXX-XXXX-XXXX"
                }
            }
        }
    ]
}



```
Guardamos los cambio de SQS. 

## 4. Vamos a S3

Creamos el bucket. ahora si que si si hemos editado bien la plantilla no nos tiene que dar ningun error y nos tiene que crear el bucket.

## 5. Vamos a SQS. 

1. Send and receive messages
2. Poll for messages

Nos deberia de aparecer un mesnaje automatico de test de S3
