# Actividades AWS CloudFormation

A continuación se detallan los ejercicios a realizar utilizando plantillas de CloudFormation.

## 1. VPC con Instancia EC2 Condicional

**Objetivo:** Crear una plantilla que implemente una VPC con una subred pública y una instancia EC2.

**Requisitos:**
*   **Conditions:** Se utilizarán para decidir si se crea la instancia basándose en una variable (parámetro) llamada `Crear Instancia` (valores `true` o `false`).
*   **Rules:** Implementar reglas para validar el tipo de instancia permitido, limitando la elección a **tres tipos** de instancias específicos.
*   **Nomenclatura (Fn::Join):** El nombre del recurso debe formarse combinando:
    1. La región.
    2. El ID de la instancia EC2.
    3. Tu nombre.
*   **Disponibilidad (GetAZs):** Utilizar `GetAZs` para obtener la zona de disponibilidad de la VPC.
*   **Etiquetado:** En la instancia EC2 se debe crear una etiqueta (Tag) con la clave `VPCID`, recuperando su valor mediante `!GetAtt`.

---

## 2. VPC Multi-Región con Mappings

**Objetivo:** Crear una VPC con subredes en diferentes regiones utilizando `Mappings`.

**Requisitos:**
*   **Mappings (CIDR):**
    *   Para una región se seleccionará el rango `10.0.0.0/20`.
    *   Para la otra región se seleccionará el rango `172.16.0.0/20`.
*   **Selección de AZ:** Se debe seleccionar dinámicamente una zona de disponibilidad utilizando `Fn::Select` y `GetAZs`.
*   **Internet Gateway Condicional:**
    *   Añadir un parámetro que tome dos valores (`true`, `false`), con `true` por defecto.
    *   Utilizar una **Condition** y `Fn::If` para decidir si se adjunta el Internet Gateway (IGW) a la VPC basándose en el parámetro anterior.

---

## 3. Application Load Balancer (ALB) y Auto Scaling Group

**Objetivo:** Crear un Application Load Balancer (ALB) y un Auto Scaling Group distribuidos en múltiples zonas de disponibilidad.

**Requisitos:**
*   **Parámetro ALBPublico:** Variable que indicará si se quiere mostrar el balanceador a Internet.
*   **Esquema del LoadBalancer:** Determinar mediante `!If` si el esquema es `internet-facing` o `internal` basándose en el parámetro anterior.
*   **Nomenclatura de recursos:**
    *   Usar un parámetro de entrada para el nombre (Valor por defecto: `vívalalandingzone`).
    *   Utilizar `Fn::Join` para unir este valor con tu propio nombre.

---

## Webgrafía y Restricciones

De todos los ejercicios se debe adjuntar la **webgrafía** utilizada.

> [!IMPORTANT]
> *   **No se puede utilizar ninguna IA.**
> *   Intenta utilizar exclusivamente la **documentación oficial de AWS**.