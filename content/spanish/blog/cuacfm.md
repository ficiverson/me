---
title: "Revamp App móvil usando Flutter en dos semanas"
meta_title: "Revamp App móvil usando Flutter en dos semanas"
description: "CUAC FM port post"
date: 2020-03-14T08:00:00Z
image: "/images/blog/cuacfm/old_iphone.png"
categories: ["Architecture"]
author: "Fernando Souto"
lang: "es"
tags: ["CUAC FM", "Android", "iOS", "Flutter", "Mobile"]
draft: false
---


Esta es la historia de cómo, en exactamente 2 semanas, realicé una renovación completa de la app **[CUAC FM](https://cuacfm.org/)** teniendo en cuenta que el tiempo de trabajo estaba limitado a las restricciones de un proyecto desarrollado en mi tiempo libre.

## Situación actual

La situación era la siguiente.

**CUAC FM** es una *radio comunitaria sin ánimo de lucro* en**A Coruña (Galicia, España)** que tenía dos aplicaciones realmente diferentes creadas en diferentes años y con dos bases de código distintas, lo que hacía realmente difícil actualizar ambas por separado, así que decidí usar el mejor kit de desarrollo móvil multiplataforma: **[Flutter](https://flutter.dev/)**.

![Android App 2017](/images/blog/cuacfm/old_android.png)

![iOS App 2018](/images/blog/cuacfm/old_iphone.png)

## Radioco

**[Radioco](http://radioco.org/es/)** es una aplicación de gestión de radio adecuada para radios comunitarias que facilita la programación, grabación en vivo y publicación.

CUAC FM está utilizando actualmente Radioco como proveedor para la generación de contenido de podcast y la programación de programas. Si te interesa este increíble software libre, visita el sitio web oficial en [http://radioco.org/es/](http://radioco.org/es/)

La nueva app móvil de **CUAC FM**  también es software libre y se llama **radiocom-flutter**. Puedes encontrar el código fuente en **[Github](https://github.com/ficiverson/radiocom-flutter)**. Siéntete libre de usarlo y recuerda: ¡PRs y comentarios son bienvenidos!

## Desarrollo y alcance 

El primer requisito era que esta app debía estar en vivo en un sprint de dos semanas (la restricción de tiempo era muy importante para llegar al aniversario de **CUAC FM** ) y el alcance inicial de la app eran los siguientes épicos:

- Transmisión en vivo
- Buscar contenido de podcast y detalle de episodios
- Reproducción de los podcasts
- Lector de noticias de la emisora y detalle de noticias
- Información adicional de la emisora, galería de imágenes, formulario de contacto, política de privacidad, licencias de software y enlaces a redes sociales

El desarrollo avanzó muy rápido y sin complicaciones ya que la app encajaba perfectamente en lo que **Flutter** nos puede ofrecer, e incluso logré agregar *notificaciones push* y soporte para *modo oscuro* en ambas plataformas. Esto es muy importante porque ambas cosas estaban planeadas para ser parte de la segunda versión.

El soporte para modo oscuro fue divertido porque en **Flutter** la integración con ambas plataformas es muy fácil, y traté de lanzar esto porque la semana pasada **WhatsApp** anunció que lo tenía y pensé: **Ok, ¡CUAC FM también!**
>  En Flutter todo es tan rápido que la iteración del producto es continua.

## Screenshots de la app 2 semanas después

![Android and iOS 2020](/images/blog/cuacfm/new_app.png)

![Android and iOS dark mode 2020](/images/blog/cuacfm/new_app_dark.png)

## Enlaces de descarga

Si quieres probar la app, sería genial recibir tus comentarios :)

Puedes probar ambas apps aquí:

[**Descarga en Google Play](https://play.google.com/store/apps/details?id=com.app.cuacfm.radio.coruna&hl=es)
[Descarga en Appstore](https://apps.apple.com/es/app/cuac-fm/id536600585)**
