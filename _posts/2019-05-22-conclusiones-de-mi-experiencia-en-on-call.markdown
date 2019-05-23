--- 
layout: post
title: Conclusiones de mi experiencia en on-call
status: publish
type: post
published: true
date: 2019-05-22 23:30
tags: 
- work
---

Durante más de 10 años he participado en rotaciones de on-call en varias empresas distintas. La misión, siempre, asegurar que uno o varios servicios web funcionan según lo esperado.

Hay muchos enfoques de on-call dependiendo de los tipos de servicios, del tamaño de la empresa, de la organización del equipo técnico, del SLA, etc... Yo he trabajado casi siempre con rotaciones de una semana, 24x7, en el que había una única persona del equipo de ingeniera en on-call.

Esto quiere decir que durante esa semana, en cualquier momento puedes recibir una alerta en el pager (hoy en día tu teléfono móvil) y tienes que ser capaz de investigar la alerta y hacer lo posible para que el servicio funcione correctamente.

Estas son conclusiones que saco yo de mis vivencias. La mayor parte de ellas me parece que aplican a cualquier contexto, pero seguro que hay cosas que se pueden y deben hacer distintas en empresas o servicios radicalmente distintos.

### La vida con el pager

Para estar en on-call necesitas poder recibir y resolver una alerta. Sin más. Esto significa tener cobertura móvil, acceso a internet y un portátil. Si puedes tener todo eso, lo mismo te da que te vayas al cine, cojas la bici, te vayas de cañas (ojo que estar de on-call ebrio tiene un extra de desafío que te la puede liar) o lo que sea.

No te obsesiones con el pager, inclúyelo en tu vida. Simplemente asegurate que para todo lo que hagas esa semana puedas, en un momento dado, cumplir con los requisitos. No hace faltar estar todo el día recluido en casa.

Eso sí, por la noche tienes que tener disciplina. Si te despierta el pager, te levantas a la primera y te pones a resolverlo. Esto no es como el despertador de por la mañana que le puedes hacer el snooze. Ten preparado el setup (preferiblemente en otro cuarto) para que no tengas que perder tiempo con tonterías. Al terminar, si no tienes que escribir un report completo por la noche, al menos apúntate todos los detalles, porque al día siguiente te acordarás de la mitad.

Esa disciplina aquí es importante porque todo por la noche va a ser más lento y torpe y además si tienes pareja has de intentar que sea lo menos molesto posible para ella. Si no prueba...

### No te montes tu propio chiringuito de alerting

Tu core de negocio son los servicios que tu desarrollas y que ofreces a tus clientes. Céntrate en hacer eso lo mejor posible. Monitorizar los servicios generalmente no es parte de ese core de negocio y que las notificaciones push, sms y llamadas lleguen al móvil de on-call menos todavía. Habrá momentos que pienses que tú lo puedes hacer mejor y más barato que otros. No te flipes, no es trivial. Como falle lo más mínimo y pase algo en tu plataforma, no te vas a enterar. Se enteraran antes tus clientes. 

Hay pocas empresas (decentes) que se dedican a esto y no han evolucionado en años porque apenas tienen competencia, pero al menos hacen el trabajo bien. Intégrate con ellos y paga por estar tranquilo.

### Prepara una estrategia clara

He visto auténticas barbaridades de estrategias de on-call por ahí. En cuanto a complejidad innecesaria, remuneración absurda, falta de training o de herramientas, gente no preparada, etc.. No hay una estrategia única para todos. Depende de cada empresa, pero montes lo que montes que esté clarinete y que todo el mundo lo entienda y lo acepte. Como la gente esté cabreada o confusa antes de empezar, imagínate cuando un día el pager le despierte 3 veces por la noche.

Cómo deberías pagar? Pues insisto, depende de cada empresa. Pero una cosa es cierta, lo que estás pagando es por la disponibilidad y la capacidad de resolver. Si luego pagas por alerta recibida te vas a encontrar con todas las trampas del mundo. La gente de on-call está haciendo concesiones en su tiempo libre por estar pendiente. Así que págales todo el tiempo que estén haciendo esas concesiones. Luego si hace falta puedes poner diferentes tipos de extras encima de eso, dependiendo de las circunstancias. Pero la base paga por la disponibilidad.

### Cuanta más gente en on-call mejor

Hay opiniones de que el on-call lo tiene que hacer el equipo de sistemas y ya. Hay que ser pobre de miras (o muy jeta) para verle sentido a eso. El on-call lo tiene que poder hacer toda persona que tenga capacidad de recibir, debugear y solucionar un problema. Cuanta más gente haya para hacer on-call menos quemarás a tu equipo, mejor entenderá todo el mundo qué significa poner cosas en producción y por dónde fallan las cosas cuando fallan y mejor conocerá todo el mundo la plataforma. 

Obviamente tienes que facilitar un training adecuado y que cualquier persona que entra a on-call posea las herramientas y conocimientos suficientes como para tener éxito la mayor parte de las veces. Para cuando no lo tengan, tienes que estar preparado para poder escalar alertas y pedir ayuda.

### Buena documentación y procesos claros

La mayor parte de las alertas son similares. Obviamente, cuando una misma alerta se repite muchas veces hay que trabajar en eliminarla, pero, de cualquier forma, es fácil que muchas tengan denominadores comunes. Prepara un buen knowledge base. Documenta claramente qué tipos de alertas pueden surgir y con qué herramientas debugearlas. Define claramente qué se espera de la persona en on-call, cuando y a quien avisar y cual es el output de la resolución de una alerta.

<img src="{{ "/images/this-is-fine.png" }}" alt="This is fine - KC Green" title="KC Green" width="600" height="300" class="aligncenter" style="border:none;"/>

### Párate a pensar durante el fuego

Obviamente, cuanto más conoces tu plataforma y más te enfrentas a fuegos, más desarrollas esa intuición de por donde van los tiros y menos "piensas" en cómo atacar el problema. Pero siempre habrá problemas que no veas tan claro desde el primer momento. Aquí, para para pensar. No te aceleres. Por grande que sea el fuego, dedica los primeros minutos a entender bien qué ocurre, a quién afecta, cuáles son las consecuencias y cómo lo puedes reproducir. Sin todo esto vas a terminar tomando decisiones aleatorias que además no vas a saber argumentar. Respira hondo, piensa y ponte en marcha.

### Organiza bien el trabajo

Habrá fuegos simples que te quites en 5 minutos y luego están los de medio equipo trabajando como locos y sudando a base de bien. Vaya noches guapas hemos pasado varias personas apagando fuegos de varias horas. En este tipo de fuegos, con varias personas involucradas, se extremamente organizado. Si puede ser, que una persona coordine los esfuerzos tanto de debugging como de resolución. Que esta persona sea la responsable también de ir comunicando al resto de equipos el estado del problema.

Si hay gente suficiente involucrada y te lo puedes permitir, ten un plan B e incluso un plan C. Habrá veces en los que aún a pesar de tener mucha información no estés seguro del tiempo o efectividad de un approach. Pero aún así encuentras valor en intentarlo. Si puedes ataca el plan A y el B al mismo tiempo y coordina a todo el mundo.

### Comunícate bien

Imaginate que en tu empresa sois 100 y que dais servicio a 1000 usuarios. Te lías a apagar un fuego importante y no dices ni mu hasta que no terminas. Cuando salgas de tu cueva va a haber una marabunta de gente con antorchas esperándote. Estar en on-call y apagar cualquier fuego en la plataforma tiene siempre el contexto de un servicio que otra gente usa. Si la plataforma tiene problemas toda esta gente está teniendo problemas. 

Casi seguro que hay más ingenieras y/o ingenieros que te pueden ayudar. Además tienes un equipo de soporte que va a saber traducir cada estado internamente al resto de la empresa y al público general. Y si eres muy enterprise-oriented tienes account managers que van a poder darles respuestas personalizadas a tus clientes más delicados. Sin contar con que toda esta gente también te va a poder dar feedback. 

Asegurate de que la comunicación fluye en todos los sentidos antes, durante y después del fuego, empezando por ti que eres quien está viendo el problema en primera mano.

### No tengas miedo a tomar decisiones

Esto debería aplicar a tu día a día en general, pero en un fuego más aún. Estás en una situación no planeada y por lo tanto vas a tener que tomar decisiones que no te has preparado. No hay una decisión mala en este momento. Obviamente no hagas nada sin haber hecho eso de pensar que digo antes. Pero teniendo todos los datos posibles, lo que no puedes es dejar las cosas a medias o pegarte años para resolver el problema porque no te atreves a tomar una decisión drástica. 

Por ejemplo, tienes afectados un 30% de los usuarios pero tienes claro que para resolver rápido el problema tienes que parar toda la maquinaria 10 minutos. Si te lo puedes permitir, igual es mejor que tires por ahí.

### Pide ayuda pero no te desentiendas

No tengas reparos en pedir ayuda si ves que tienes dificultades para entender o resolver un problema. Una estrategia de on-call no debería estar hecha para evaluar a nadie o competir con tus colegas. Está hecha para resolver problemas. 

Pide ayuda. Pero ojo, cuando pidas ayuda o escales el problema no des el trabajo por hecho y te vayas de cañas. Tú has de ser el responsable porque el problema se resuelve y se toman las medidas necesarias para que todo acabe bien. Te van a ayudar, pero tu vas a estar al pie del cañón hasta que la cosa termine. Eres el último en irse a la cama.

Si mantienes un sistema de monitorización saludable, tienes una estrategia de on-call clara y justa, los beneficios de estar de on-call superan claramente a las desventajas. Además cuando te toca on-call durante la comida de Navidad te conviertes en el alma de la fiesta. Todo suma.
