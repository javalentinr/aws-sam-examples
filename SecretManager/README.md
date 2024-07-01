# Crear secret manager como infrastructura como codigo en SAM AWS.


## Prerequisitos

- AWS CLI instalada y  configurada
- AWS SAM CLI instalada
- Node.js and npm instalado depende la versiona usar.


### Explicación

1. **Instalación de AWS Lambda Powertools**: Instalamos las bibliotecas de Powertools para Node.js (En el caso que no se tenga en el proyecto).

2. **Modificación de `template.yaml`**: 
 Para esto vamos al template.yaml de nuestro proyecto. 
 
## Archivo `template.yaml`

El archivo `template.yaml` define los recursos de AWS que se crearán utilizando AWS SAM. En este caso, vamos a crear un secreto en AWS Secrets Manager.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: >
  SAM template for creating a Secrets Manager secret and a Lambda function that uses the secret.

Parameters:
  DBHost:
    Type: String
    Description: Host de la base de datos
  DBUser:
    Type: String
    Description: Usuario de la base de datos
  DBPassword:
    Type: String
    Description: Contraseña de la base de datos
    NoEcho: true
  DBPort:
    Type: String
    Description: Puerto de la base de datos
  DBDatabase:
    Type: String
    Description: Nombre de la base de datos
    NoEcho: true

Resources:
  DatabaseConnectionSecret:
    Type: AWS::SecretsManager::Secret
    DeletionPolicy: Delete
    UpdateReplacePolicy: Delete
    Properties:
      Description: Box sdk config variables
      SecretString:
        Fn::ToJsonString:
          connectionDetails:
            host: !Ref DBHost
            user: !Ref DBUser
            password: !Ref DBPassword
            port: !Ref DBPort
            database: !Ref DBDatabase

  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs14.x
      CodeUri: src/functions/myFunction/
      Environment:
        Variables:
          SECRET_NAME: !Ref DatabaseConnectionSecret
      Policies:
        - AWSLambdaBasicExecutionRole
        - Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - secretsmanager:GetSecretValue
              Resource: !Ref DatabaseConnectionSecret

Outputs:
  SecretArn:
    Description: The ARN of the created secret
    Value: !Ref DatabaseConnectionSecret
```

### Explicación del Código

-Resources: Sección donde se definen los recursos que se crearán.

- DatabaseConnectionSecret: Nombre lógico del recurso. Este nombre se utiliza para referirse al recurso dentro de la plantilla.

- Type: Tipo de recurso, en este caso AWS::SecretsManager::Secret.

- DeletionPolicy: Política de eliminación. Delete significa que el secreto se 
eliminará cuando se elimine la pila de CloudFormation.

- UpdateReplacePolicy: Política de reemplazo. Delete significa que el secreto se eliminará si se reemplaza por una actualización.

- Properties: Propiedades del secreto.

- Description: Descripción del secreto.

- SecretString: Cadena JSON que contiene los detalles del secreto.

- Fn::ToJsonString: Función que convierte un objeto en una cadena JSON.

- connectionDetails: Objeto que contiene los detalles de la conexión.

- host: Referencia a un parámetro llamado DBHost.

- user: Referencia a un parámetro llamado DBUser.

- password: Referencia a un parámetro llamado DBPassword.

- port: Referencia a un parámetro llamado DBPort.

- database: Referencia a un parámetro llamado DBDatabase.