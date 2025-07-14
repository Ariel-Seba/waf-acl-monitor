# AWS WAF ACL Monitor (EventBridge + CloudTrail)

## Descripción

Este proyecto monitorea en tiempo real los cambios realizados en las AWS WAF ACLs (Web ACLs) de tu cuenta. Utiliza eventos de CloudTrail y EventBridge para detectar cambios, guarda el historial en DynamoDB y envía alertas detalladas por SNS, incluyendo exactamente qué reglas fueron agregadas, eliminadas o modificadas.

---

## ¿Cómo funciona?

1. **CloudTrail** registra todos los eventos de cambios en WAF ACLs (`CreateWebACL`, `UpdateWebACL`, `DeleteWebACL`).
2. **EventBridge** detecta estos eventos y dispara la función Lambda.
3. **La Lambda**:
   - Extrae el usuario responsable (incluyendo federados SSO), el tipo de evento, el nombre y el ID del ACL.
   - Lee de DynamoDB el estado anterior de las reglas de ese ACL.
   - Compara las reglas anteriores y nuevas, y detecta:
     - Reglas agregadas
     - Reglas eliminadas
     - Reglas modificadas
   - Guarda el nuevo estado en DynamoDB para futuras comparaciones.
   - Envía una alerta SNS con el resumen del cambio y el usuario responsable.

---

## Ejemplo de alerta recibida

```
🚨 Cambio detectado en AWS WAF ACL

Tipo de evento: UpdateWebACL
ACL: test-1 (ID: b824c8a9-b9d9-45b9-8460-12bcb4ae9ec9, Scope: REGIONAL)
ARN: arn:aws:wafv2:us-east-1:123456789012:regional/webacl/test-1/b824c8a9-b9d9-45b9-8460-12bcb4ae9ec9
Usuario responsable: federado: ariel.seba@cloudhesive.com (rol: AWSReservedSSO_AWSAdministratorAccess_91070a25d0c2f403)
Tipo de usuario: AssumedRole
Timestamp: 2025-07-14T21:15:37Z
Cuenta: 882035033547
Región: us-east-1

Cambios detectados en las reglas:
- Regla agregada: AWS-AWSManagedRulesAmazonIpReputationList
- Regla modificada: AWS-AWSManagedRulesCommonRuleSet
- Regla eliminada: MiReglaPersonalizada
```

---

## Arquitectura

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ CloudTrail   │    │ EventBridge  │    │   Lambda     │
│ (API Calls)  │──▶│ (Rule)       │──▶│ (Monitor)    │
└──────────────┘    └──────────────┘    └─────┬────────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │   SNS Topic  │
                                       └──────────────┘
                                              │
                                              ▼
                                       ┌──────────────┐
                                       │ DynamoDB     │
                                       │ (Historial)  │
                                       └──────────────┘
```

---

## Despliegue

1. **Clona el repositorio y edita parámetros si lo deseas.**
2. **Despliega el stack con CloudFormation:**

```bash
aws cloudformation deploy \
  --stack-name waf-acl-monitor \
  --template-file cloudformation/waf-acl-monitor.yml \
  --capabilities CAPABILITY_IAM
```

3. **Suscríbete al SNS Topic** para recibir alertas por email.

---

## Parámetros principales

- `Region`: Región AWS donde se desplegarán los recursos.
- `LogRetentionDays`: Días de retención de logs de CloudWatch.
- `DynamoDBTableName`: Nombre de la tabla DynamoDB para historial de cambios.

---

## ¿Qué detecta?

- Cambios en reglas de cualquier ACL (agregadas, eliminadas, modificadas)
- Usuario responsable (IAM, federado SSO, etc.)
- Tipo de evento (creación, actualización, borrado)
- Historial completo en DynamoDB

---

## ¿Cómo se almacena el historial?

- Cada vez que se detecta un cambio, se guarda el evento completo y el nuevo estado de las reglas en DynamoDB.
- El historial permite auditar todos los cambios y ver el “diff” de reglas entre versiones.

---

## Requisitos

- AWS CloudTrail habilitado
- Permisos para crear recursos: Lambda, SNS, DynamoDB, EventBridge, IAM

---

## Personalización

Puedes modificar el mensaje de alerta, los campos guardados en DynamoDB o los eventos monitoreados editando el template y la función Lambda.

---

## Soporte

¿Dudas o mejoras? ¡Abre un issue o contacta al autor! 