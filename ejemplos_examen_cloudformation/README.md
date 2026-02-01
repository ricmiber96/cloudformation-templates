# ☁️ Guía de Estudio CloudFormation - Examen UD4

Este documento resume las 10 secciones de una plantilla de CloudFormation, explicadas con ejemplos prácticos y trucos para el examen.

---

## 📑 Índice de Secciones
1. [AWSTemplateFormatVersion](#1-awstemplateformatversion)
2. [Description](#2-description)
3. [Metadata](#3-metadata)
4. [Parameters](#4-parameters)
5. [Rules](#5-rules)
6. [Mappings](#6-mappings)
7. [Conditions](#7-conditions)
8. [Transform](#8-transform)
9. [Resources](#9-resources)
10. [Outputs](#10-outputs)

---

### 1. AWSTemplateFormatVersion
Define la capacidad de la plantilla.
*   **Truco de Examen:** Solo existe una fecha válida. Si te preguntan por otra, es trampa.
*   **Código:**
    ```yaml
    AWSTemplateFormatVersion: '2010-09-09'
    ```

### 2. Description
Cadena de texto para documentar la plantilla.
*   **Truco:** Debe ir obligatoriamente después de la versión.
*   **Código:**
    ```yaml
    Description: "Plantilla para desplegar entorno LAMP en examen."
    ```

### 3. Metadata
Información adicional no procesada directamente por el motor de despliegue, pero usada por herramientas externas o la consola.
*   **Uso Clave 1 (Interface):** Agrupar y ordenar parámetros en la consola de AWS (para que no salgan desordenados).
*   **Uso Clave 2 (Init):** Definir configuraciones de software dentro de instancias EC2 (`cfn-init`).
*   **Código (Interface):**
    ```yaml
    Metadata:
      AWS::CloudFormation::Interface:
        ParameterGroups:
          - Label: { default: "Configuración de Red" }
            Parameters: [VpcCidr, SubnetCidr]
    ```

### 4. Parameters
Valores que introduces al crear la pila (Stack) para hacer la plantilla dinámica.
*   **Propiedades clave:** `Default`, `AllowedValues` (lista desplegable), `Type` (String, Number, List...), `NoEcho` (para ocultar passwords).
*   **Código:**
    ```yaml
    Parameters:
      Environment:
        Type: String
        Default: Dev
        AllowedValues: [Dev, Prod]
        Description: "Entorno de despliegue"
      DBPassword:
        Type: String
        NoEcho: true # ¡Importante para seguridad!
    ```

### 5. Rules
Validaciones lógicas de los parámetros **antes** de intentar crear recursos.
*   **Diferencia con Conditions:** *Rules* valida inputs (lanza error si falla). *Conditions* decide si crear o no recursos.
*   **Ejemplo:** "Si es Producción, no permitas instancias t2.nano".
*   **Código:**
    ```yaml
    Rules:
      CheckInstanceSize:
        RuleCondition: !Equals [!Ref Environment, "Prod"]
        Assertions:
          - Assert: !Not [!Equals [!Ref InstanceType, "t2.nano"]]
            AssertDescription: "En Prod no uses t2.nano"
    ```

### 6. Mappings
Tablas de consulta estáticas (Key-Value). Funcionan como un diccionario/hashmap.
*   **Uso típico:** Asignar AMIs por Región o Tamaños de instancia por Entorno.
*   **Función Intrínseca:** Se accede con `!FindInMap [NombreMapa, TopKey, SecondKey]`.
*   **Código:**
    ```yaml
    Mappings:
      RegionMap:
        us-east-1: { AMI: "ami-0ff8a91507f77f867" }
        eu-west-1: { AMI: "ami-0a584ac55a7631c0c" }
    ```

### 7. Conditions
Lógica Booleana (True/False) basada en parámetros.
*   **Uso:** Controla si un recurso se crea (`Condition: MiCondicion`) o cambia el valor de una propiedad (`!If`).
*   **Funciones:** `!Equals`, `!And`, `!Or`, `!Not`.
*   **Código:**
    ```yaml
    Conditions:
      IsProd: !Equals [!Ref Environment, "Prod"]

    Resources:
      MyProdBucket: 
        Type: AWS::S3::Bucket
        Condition: IsProd  # Solo se crea si es Prod
    ```

### 8. Transform
Permite usar Macros y extensiones del lenguaje, principalmente SAM (Serverless Application Model).
*   **Truco:** Necesario para definir recursos como `AWS::Serverless::Function`. Simplifica la definición de Lambdas y APIs.
*   **Código:**
    ```yaml
    Transform: AWS::Serverless-2016-10-31
    ```

### 9. Resources
**LA SECCIÓN OBLIGATORIA.** Define los componentes de infraestructura.
*   **Estructura:** ID Lógico -> `Type` -> `Properties`.
*   **Truco:** Debes saber usar `!Ref` (para IDs o Parámetros) y `!GetAtt` (para atributos como IPs o Endpoints).
*   **Código:**
    ```yaml
    Resources:
      MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
          ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
          InstanceType: t2.micro
    ```

### 10. Outputs
Valores que devuelve la pila al terminar.
*   **Uso:** Mostrar una IP, una URL, o compartir datos con otras pilas.
*   **Importante:** `Export` permite que otra pila use `!ImportValue` para leer este dato (Cross-stack reference).
*   **Código:**
    ```yaml
    Outputs:
      WebUrl:
        Description: URL del servidor
        Value: !Sub "http://${MyEC2Instance.PublicDnsName}"
        Export:
          Name: !Sub "${AWS::StackName}-URL"
    ```

---

## ⚡ Funciones Intrínsecas (Cheat Sheet)

Son funciones que se usan *dentro* de Resources y Outputs. ¡Imprescindibles!

| Función | Sintaxis Corta | Para qué sirve |
| :--- | :--- | :--- |
| **Ref** | `!Ref RecursoOParametro` | Devuelve el valor de un parámetro o el ID físico de un recurso. |
| **GetAtt** | `!GetAtt Recurso.Atributo` | Obtiene un dato específico (ej. `MyEC2.PublicIp`, `MyRDS.Endpoint.Address`). |
| **Sub** | `!Sub "Hola ${Parametro}"` | Sustituye variables dentro de una cadena de texto (interpolación). |
| **Join** | `!Join [",", [A, B, C]]` | Une una lista de valores con un delimitador ("A,B,C"). |
| **Select** | `!Select [0, Lista]` | Elige un elemento de una lista por su índice. |
| **GetAZs** | `!GetAZs ""` | Devuelve la lista de Zonas de Disponibilidad de la región actual. |
| **FindInMap**| `!FindInMap [Mapa, K1, K2]` | Busca valores en la sección Mappings. |
| **ImportValue**| `!ImportValue NombreExport` | Lee un Output exportado por **otra** pila. |

---

## 🛠 Comandos CLI Básicos

Recuerda estos comandos vistos en el temario:

1.  **Validar sintaxis:**
    `aws cloudformation validate-template --template-body file://plantilla.yaml`
2.  **Crear pila:**
    `aws cloudformation create-stack --stack-name MiPila --template-body file://plantilla.yaml`
3.  **Actualizar pila:**
    `aws cloudformation update-stack --stack-name MiPila --template-body file://plantilla.yaml`
4.  **Borrar pila:**
    `aws cloudformation delete-stack --stack-name MiPila`