# Documentación de OpenAPI

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)

## Introducción
Esta documentación está destinada a guiarte a través del proceso de comprensión, configuración de la documentación de la lambda con openAPI.

Para esto necesitamos crear un archivo de 
openapi.yaml
Para esto ya se tiene una plantilla predifinida la manera de meter tus documentación de tu lambda es mas facil la estructura de de la actual en ejemplo es :

``` yaml
openapi: 3.0.0
info:
  title: Nombre de tu API
  description: Descripción breve de tu API
  version: 1.0.0
servers:
  - url: https://api.tu-dominio.com
paths:
  /ruta-de-tu-lambda:
```

un ejemplo documentador una lambda generica

``` yaml
openapi: 3.0.0
info:
  title: Mi API de Lambda
  description: Documentación de la API de mi Lambda utilizando OpenAPI
  version: 1.0.0
servers:
  - url: https://api.mi-dominio.com
paths:
  /mi-lambda:
    get:
      summary: Obtener información de mi Lambda
      description: Este endpoint devuelve información sobre mi Lambda.
      responses:
        '200':
          description: Respuesta exitosa
          content:
            application/json:
              schema:
                type: object
                properties:
                  mensaje:
                    type: string
                    example: 'Respuesta exitosa'
        '400':
          description: Solicitud incorrecta
        '500':
          description: Error interno del servidor

```


En la parte de paths es donde tenemos que poner el path que registramos en nuestro template

summary: es donde ponemos una breve descripción para saber que es lo que hace nuestra lambda en pocas palabras.

operationId: en este caso es una manera de darle un identificador a la lambda que se esta documentando.

description: Es donde ponemos la descripción mas detallada para saber que hacer nuestra lambda.

tags: Son las etiquetas con la cual podemos separar las documentaciones por ejemplo todas las lambdas sde catalogos podemos crear un tag llamado 'Catalogo' con esto nos podra facilitar la busqueda por su stack de lambdas de ese tag.

responses: cuando vamos a responder aqui ponemos toda la parte de code estatus errores 200,201,400,404,500. y de describen un ejemplo de un 200 

``` yaml
responses:
        '200':
          description: 'Success'
          content:
            application/json:
              schema:
                type: object
                properties:
                  message:
                    type: string
                    example: "successfully"
                  data:
                    type: array
                    minItems: 0
                    maxItems: 100
                    items:
                      type: object
                      properties:
                        value:
                          type: number
                          example: 2
                        label:
                          type: string
                          example: "NISSAN"
                    example:
                      - value: 2
                        label: "NISSAN"
                      - value: 51
                        label: "CHEVROLET2"
```