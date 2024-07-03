# Documentación de Función AWS Lambda

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)
3. [Estructura del Proyecto](#estructura-del-proyecto)
4. [Función Lambda](#función-lambda)
5. [Módulos Compartidos](#módulos-compartidos)
6. [Creación de template](#creacion-template)
7. [Despliegue en AWS](#despliegue-en-aws)

## Introducción
Esta documentación está destinada a guiarte a través del proceso de comprensión, configuración y despliegue de una función AWS Lambda. AWS Lambda te permite ejecutar código sin aprovisionar o gestionar servidores. Solo pagas por el tiempo de cómputo que consumes.

En este ejemplo, crearemos una función Lambda que se conecta a una base de datos MySQL, ejecuta un procedimiento almacenado y devuelve los resultados a través de un API Gateway.

## Requisitos Previos
Antes de comenzar, asegúrate de tener lo siguiente:
- Node.js y npm instalados
- AWS CLI configurado con tus credenciales
- Acceso a una cuenta de AWS
- Base de datos MySQL con el procedimiento almacenado `sp_select_cmb_activo_condicion_pieza_cotizacion()`

## Estructura del Proyecto
Tu proyecto debe tener la siguiente estructura:

```
WEBBESTABACK/
├── .aws-sam/
├── .github/
├── amplify/
├── docs/
├── node_modules/
├── portman/
├── src/
│   ├── functions/
│   │   └── index.mjs
│   └── shared/
│       ├── apigateway/
│       │   └── index.mjs
│       ├── database/
│       │   └── index.mjs
│       └── lambda-powertools/
│           └── index.mjs
├── .c8rc.json
├── .eslintrc.json
├── .gitignore
├── .spectral.yaml
├── openapi.yaml
├── package-lock.json
├── package.json
├── README.md
├── samconfig.ci.yaml
├── template.yaml
└── vitest.config.mjs
```

## Función Lambda
Aquí tienes el código de la función Lambda ubicada en `src/functions/index.mjs`:

```javascript
import { getResponse } from '../shared/apigateway/index.mjs';
import { createConnection, executeQuery } from '../shared/database/index.mjs';
import { initializePowertools, logger } from '../shared/lambda-powertools/index.mjs';

export const handler = initializePowertools(async () => {
  let mysqlConnection;

  try {
    mysqlConnection = await createConnection();
    const data = await getConditionPiece(mysqlConnection);
    if (!data || data.length === 0) {
      logger.warn('Not found.');
      return getResponse(404, {
        'message': 'Not found.',
        'data': {}
      });
    }

    return getResponse(200, {
      data: mapResponse(data)
    });
  } catch (error) {
    logger.error('Internal server error:', error);

    return getResponse(500, {
      message: 'Something went wrong!',
      data: {}
    });
  } finally {
    await mysqlConnection?.end();
  }
});

export const getConditionPiece = async (connection) => {
  const response = await executeQuery(connection, `CALL sp_select_cmb_activo_condicion_pieza_cotizacion()`);
  return response;
};

export const mapResponse = (data) => {
  const transformedResults = data.map((item) => ({
    value: item.nIdActivoCondPiezaCot,
    label: item.sNombre
  }));
  return transformedResults;
};
```

### Explicación

Imports: La función importa funciones auxiliares de los módulos compartidos.

Handler: La función principal (handler) está envuelta con initializePowertools para el registro y monitoreo.

Conexión a la Base de Datos: Crea una conexión a la base de datos MySQL.

Ejecución de la Consulta: Ejecuta un procedimiento almacenado para obtener datos.

Manejo de Respuestas: Basado en los datos obtenidos, devuelve respuestas HTTP adecuadas.

Limpieza: Asegura que la conexión a la base de datos se cierre correctamente.

## Creación Template

### Definición de Función Lambda en template.yaml

La siguiente sección del archivo `template.yaml` define una función Lambda llamada `GetConditionPiece`. Esta función se configura con las propiedades y eventos necesarios para su correcta ejecución dentro de AWS. A continuación, se presenta una explicación detallada de cada componente:

```yaml
GetConditionPiece:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/functions/getConditionPiece
    Handler: index.handler
    Environment:
      Variables:
        DATABASE_CONNECTION_SECRET: !Ref DatabaseConnectionSecret
    Events:
      GetStatusFlowEvent:
        Type: Api
        Properties:
          RestApiId: !Ref API
          Path: /pieza-condicion
          Method: GET
    Policies:
      - AWSLambdaBasicExecutionRole
      - Version: 2012-10-17
        Statement:
          - Effect: Allow
            Action:
              - secretsmanager:GetSecretValue
            Resource:
              - !Ref DatabaseConnectionSecret
  Metadata:
    BuildMethod: esbuild
    BuildProperties:
      Format: esm
      Minify: false
      OutExtension:
        - .js=.mjs
      Target: es2020
      Sourcemap: true
      EntryPoints:
        - index.mjs
      Banner:
        - js=import { createRequire } from 'module'; const require = createRequire(import.meta.url);
```

### Explicación de las Propiedades

- **Type**: Especifica que se trata de una función sin servidor (AWS::Serverless::Function).
- **Properties**:
  - **CodeUri**: Ruta al código fuente de la función Lambda.
  - **Handler**: Especifica el método de manejo de la función Lambda.
  - **Environment**:
    - **Variables**:
      - **DATABASE_CONNECTION_SECRET**: Variable de entorno que contiene la referencia al secreto de conexión a la base de datos.
  - **Events**:
    - **GetStatusFlowEvent**: Evento que activa la función Lambda.
      - **Type**: Tipo de evento, en este caso, una API.
      - **Properties**:
        - **RestApiId**: Referencia al ID de la API REST.
        - **Path**: Ruta del endpoint que activará la función.
        - **Method**: Método HTTP (GET) que activará la función.
  - **Policies**: Políticas de IAM necesarias para la función.
    - **AWSLambdaBasicExecutionRole**: Rol básico de ejecución de Lambda.
    - **Version**: Versión de la política.
    - **Statement**:
      - **Effect**: Efecto de la política (Allow).
      - **Action**: Acciones permitidas (secretsmanager:GetSecretValue).
      - **Resource**: Recurso sobre el que se aplica la acción.
- **Metadata**:
  - **BuildMethod**: Método de construcción (esbuild).
  - **BuildProperties**:
    - **Format**: Formato del módulo (esm).
    - **Minify**: Indica si el código debe ser minimizado (false).
    - **OutExtension**: Extensión de salida (.mjs).
    - **Target**: Versión del ECMAScript (es2020).
    - **Sourcemap**: Indica si debe generarse un mapa de código fuente (true).
    - **EntryPoints**: Puntos de entrada del código.
    - **Banner**: Código que se añade al principio del archivo generado.

