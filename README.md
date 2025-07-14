# AWS WAF ACL Monitor

Sistema de monitoreo y alertas para cambios en AWS WAF Access Control Lists (ACLs) usando CloudFormation, Lambda y SNS.

## 🎯 Descripción

Este proyecto proporciona una solución completa para monitorear cambios en AWS WAF ACLs y enviar alertas automáticas cuando se detectan modificaciones. Es especialmente útil para:

- **Seguridad**: Detectar cambios no autorizados en configuraciones de WAF
- **Compliance**: Mantener auditoría de cambios en reglas de seguridad
- **Operaciones**: Notificar al equipo sobre modificaciones en ACLs
- **Debugging**: Rastrear cambios que puedan afectar el comportamiento del WAF

## ✨ Características

- 🔍 **Monitoreo en tiempo real** de ACLs regionales y globales
- 📊 **Detección detallada** de cambios en reglas, rate limits, statements
- 📧 **Alertas configurables** via SNS con detalles específicos
- 💾 **Historial completo** en DynamoDB con estados anteriores
- ⚙️ **Configuración flexible** de intervalos de monitoreo
- 🛡️ **Permisos mínimos** siguiendo el principio de menor privilegio

## 🏗️ Arquitectura

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CloudWatch    │    │     Lambda      │    │      SNS        │
│   EventBridge   │───▶│   Function      │───▶│     Topic       │
│   (Trigger)     │    │   (Monitor)     │    │   (Alerts)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │    DynamoDB     │
                       │   (History)     │
                       └─────────────────┘
```

## 📋 Prerrequisitos

- AWS CLI configurado con permisos adecuados
- Cuenta de AWS con acceso a:
  - CloudFormation
  - Lambda
  - SNS
  - DynamoDB
  - WAFv2
  - CloudWatch
  - IAM

## 🚀 Instalación

### 1. Clonar el repositorio

```bash
git clone <tu-repositorio>
cd WAF
```

### 2. Desplegar el stack de CloudFormation

```bash
aws cloudformation create-stack \
  --stack-name waf-acl-monitor \
  --template-body file://cloudformation/waf-acl-monitor.yml \
  --parameters ParameterKey=MonitoringInterval,ParameterValue=5 \
               ParameterKey=Region,ParameterValue=us-east-1 \
               ParameterKey=LogRetentionDays,ParameterValue=30 \
               ParameterKey=DynamoDBTableName,ParameterValue=waf-acl-monitor-history \
  --capabilities CAPABILITY_IAM
```

### 3. Verificar el despliegue

```bash
aws cloudformation describe-stacks --stack-name waf-acl-monitor
```

## ⚙️ Configuración

### Parámetros de CloudFormation

| Parámetro | Descripción | Valor por defecto | Opciones |
|-----------|-------------|-------------------|----------|
| `MonitoringInterval` | Intervalo de monitoreo en minutos | 5 | 1, 5, 10, 30, 60 |
| `Region` | Región AWS para el despliegue | us-east-1 | Regiones soportadas |
| `LogRetentionDays` | Días de retención de logs | 30 | 1-365 |
| `DynamoDBTableName` | Nombre de la tabla DynamoDB | waf-acl-monitor-history | Cualquier nombre válido |

### Variables de entorno de Lambda

- `SNS_TOPIC_ARN`: ARN del topic SNS para alertas
- `DYNAMODB_TABLE`: Nombre de la tabla DynamoDB
- `REGION`: Región AWS
- `MONITORING_INTERVAL`: Intervalo de monitoreo

## 📊 Monitoreo

### Logs de CloudWatch

Los logs se almacenan en: `/aws/waf/acl-monitor/<stack-name>`

### Métricas disponibles

- **WAFACLChanges**: Número de cambios detectados
- **Lambda Duration**: Tiempo de ejecución de la función
- **Lambda Errors**: Errores en la ejecución

### Alertas SNS

Las alertas incluyen:
- Tipo de cambio (agregado, eliminado, modificado)
- Detalles específicos de las reglas afectadas
- Timestamp del cambio
- Información de la cuenta y región

## 🔧 Funcionalidades

### Detección de cambios

El sistema detecta cambios en:

- ✅ **ACLs agregadas/eliminadas**
- ✅ **Reglas agregadas/eliminadas**
- ✅ **Modificaciones en rate limits**
- ✅ **Cambios en statements de reglas**
- ✅ **Modificaciones en acciones**
- ✅ **Cambios en prioridades**
- ✅ **Modificaciones en DefaultAction**
- ✅ **Cambios en descripciones**
- ✅ **Modificaciones en VisibilityConfig**

### Historial en DynamoDB

Cada ejecución guarda:
- Estado anterior y actual
- Tipo de cambio detectado
- Detalles específicos
- Timestamp de la ejecución

## 🛠️ Mantenimiento

### Actualizar el stack

```bash
aws cloudformation update-stack \
  --stack-name waf-acl-monitor \
  --template-body file://cloudformation/waf-acl-monitor.yml \
  --parameters ParameterKey=MonitoringInterval,ParameterValue=5 \
               ParameterKey=Region,ParameterValue=us-east-1 \
               ParameterKey=LogRetentionDays,ParameterValue=30 \
               ParameterKey=DynamoDBTableName,ParameterValue=waf-acl-monitor-history
```

### Eliminar el stack

```bash
aws cloudformation delete-stack --stack-name waf-acl-monitor
```

## 🔍 Troubleshooting

### Problemas comunes

1. **Error de permisos**: Verificar que la cuenta tenga permisos para WAFv2
2. **Lambda timeout**: Aumentar el timeout si hay muchos ACLs
3. **Errores de serialización**: Ya resuelto con la función `clean_for_json`

### Verificar logs

```bash
aws logs tail /aws/waf/acl-monitor/waf-acl-monitor --follow
```

## 📝 Estructura del proyecto

```
WAF/
├── cloudformation/
│   └── waf-acl-monitor.yml    # Template principal
├── README.md                  # Documentación
└── .gitignore                # Archivos a ignorar
```

## 🤝 Contribuciones

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver el archivo `LICENSE` para más detalles.

## 🆘 Soporte

Para soporte técnico o preguntas:
- Crear un issue en el repositorio
- Contactar al equipo de desarrollo

## 🔄 Changelog

### v1.0.0
- ✅ Monitoreo básico de ACLs
- ✅ Detección de cambios en reglas
- ✅ Alertas via SNS
- ✅ Historial en DynamoDB
- ✅ Configuración flexible
- ✅ Manejo de errores mejorado 