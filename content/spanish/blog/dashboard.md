---
title: "Cómo construir un panel en Azure Cloud utilizando consultas de App Insights con KQL generadas por un LLM"
meta_title: "Cómo construir un panel en Azure Cloud utilizando consultas de App Insights con KQL generadas por un LLM"
description: "Cómo construir un panel en Azure Cloud utilizando consultas de App Insights con KQL generadas por un LLM"
date: 2024-12-22T05:00:00Z
image: "/images/blog/dashboard/dashboard.jpeg"
categories: ["Dashboard"]
author: "Fernando Souto"
lang: "es"
tags: ["dashboard", "azure","LLM"]
draft: false
---



Construir un panel robusto y útil en Azure Application Insights con KQL (Kusto Query Language) permite a los equipos de desarrollo monitorear y analizar el rendimiento de su aplicación y el comportamiento de los usuarios. Esta guía te llevará a través de la creación de un panel con ejemplos de KPIs y gráficos correspondientes. No sé nada sobre KQL, pero usaré una LLM para generar las consultas que necesito.

## ¿Por qué usar App Insights y KQL?

Azure Application Insights es una herramienta poderosa para rastrear la salud, el uso y el rendimiento de tus aplicaciones. Combinado con KQL, proporciona una flexibilidad incomparable para consultar y visualizar datos. Al final de esta guía, tendrás un panel claro y accionable con métricas como el número de usuarios, mensajes y usuarios activos vs. inactivos.

## Configuración de tu panel

**Accede a Application Insights:** Navega a tu resource de Application Insights en el portal de Azure.

**Abre la herramienta de Logs (Analytics):** Aquí es donde escribirás tus consultas KQL.

**Crea métricas y visualizaciones:** Escribe consultas para los KPIs, luego guárdalas como partes de un workbook compartido o dashboard.

## Enviar user_Id a App Insights

**Configura la telemetría en tu aplicación:**

Para aplicaciones que usan el SDK de Application Insights, enriquece los datos de telemetría configurando el contexto de user_Id.

Ejemplo para .NET:
```java
var telemetryClient = new TelemetryClient();
telemetryClient.Context.User.Id = "user123";
telemetryClient.TrackTrace("Sample trace with user ID");
```
Ejemplo para JavaScript:
```java
appInsights.setAuthenticatedUserContext("user123"); 
appInsights.trackTrace({ message: "Sample trace with user ID" });
```

**Verifica los datos:**

Revisa tus trazas en Application Insights para confirmar que ```user_Id``` se está registrando.

**Modifica tus consultas KQL:**

Una vez que ```user_Id``` esté disponible en las trazas, puedes usarlo en tus consultas para analizar datos específicos de los usuarios.

## Principales KPIs a incluir en tu panel

### Usuarios totales

Esta métrica te da el conteo de usuarios únicos que interactúan con tu aplicación.

**Query:**
```java
traces
| where timestamp >= startofday(now() - 30d)
| summarize TotalUsers = dcount(user_Id)
```

**Chart:**  Muestra como un solo número o en un gráfico de columnas mostrando tendencias a lo largo del tiempo.

### Mensajes totales

El número total de mensajes enviados en tu aplicación.

**Query:**
```java
traces
| where timestamp >= startofday(now() - 30d)
| summarize TotalMessages = count()
```

**Chart:** Número total o un gráfico de tiempo para mostrar tendencias.

### Top usuarios por número de mensajes

Identifica a los usuarios más activos según el número de mensajes que enviaron.

**Query:**
```java
traces
| where timestamp >= startofday(now() - 30d)
| summarize MessageCount = count() by user_Id
| top 10 by MessageCount desc
```

**Chart:** Gráfico de barras mostrando los 10 principales usuarios y sus mensajes acumulados.

### Mensajes por día

Rastrea el volumen de mensajes enviados diariamente para identificar tendencias de uso.

**Query:**
```java
traces
| where timestamp >= startofday(now() - 30d)
| summarize DailyMessages = count() by bin(timestamp, 1d)
```

**Chart:** Gráfico de líneas o columnas para mostrar la actividad diaria.

### Usuarios activos vs. inactivos

Analiza el engagement de los usuarios categorizándolos como activos (que enviaron al menos un mensaje) o inactivos.

**Query:**
```java
let ActiveUsers = traces
    | where timestamp >= startofday(now() - 30d)
    | summarize ActiveUserCount = dcount(user_Id);
let TotalUsers = traces
    | summarize TotalUserCount = dcount(user_Id);
union (ActiveUsers | extend Category = 'Active')
     (TotalUsers | extend Category = 'Inactive', Count = TotalUserCount - ActiveUserCount)
```

**Chart:** Gráfico de torta mostrando la proporción de usuarios activos vs. inactivos.

## Construir el panel

**Agregar consultas al workbook:** Guarda cada consulta como una ficha separada en un workbook de Application Insights.

**Personalizar visualizaciones:** Ajusta los tipos de gráficos y los diseños para hacer que el panel sea intuitivo.

**Configurar alertas:** Configura alertas basadas en umbrales para cualquier KPI, como una caída repentina de usuarios activos.

## Ejemplo de salida del output

A continuación, se muestran ejemplos de gráficos generados con las consultas:

**Mensajes diarios** (Bar chart)

![Daily messages per unidque users](/images/blog/dashboard/dashboard3.png)

**Top usuarios por número de mensajes** Count (Pie chart)

![Top users](/images/blog/dashboard/dashboard4.png)

**Tendencia de mensajes a lo largo del tiempo** (Time series)

![Message trend every hour](/images/blog/dashboard/dashboard1.png)

**Tendencia de usuarios a lo largo del tiempo** (Area chart)

![User trend](/images/blog/dashboard/dashboard2.png)

## Lecciones aprendidas

**Usar LLM para generar consultas:** Automatiza la creación de consultas usando una LLM. La LLM es bastante buena creando estas consultas para ti.

**Limpieza de datos:** Asegúrate de que tus registros contengan datos limpios y consistentes para evitar malas interpretaciones.

**Eficiencia:** Usa funciones de resumen como summarize y dcount para optimizar las consultas.

**Compromiso de usuarios:** Monitorear usuarios activos vs. inactivos es crucial para estrategias de retención.

**Visualización:** Elige los tipos de gráficos cuidadosamente para maximizar la claridad e impacto.

**Automatización:** Automatiza la actualización de tu panel para que siempre refleje los datos más recientes.

## Conclusión

Construir un panel con Azure Application Insights y KQL otorga a los equipos información accionable. Al enfocarse en KPIs relevantes como la actividad de los usuarios y las tendencias de uso, puedes comprender mejor el rendimiento de tu aplicación y el comportamiento de los usuarios. Incorporar las lecciones aprendidas asegura que tu panel sea efectivo y sostenible.

Usa la LLM para generar las consultas KQL y empieza a crear tu propio panel hoy y desbloquea el potencial de los datos de tu aplicación.
