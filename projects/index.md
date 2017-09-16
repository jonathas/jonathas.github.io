---
layout: page
title: "Projects"
excerpt: "Some of the projects I've developed or am currently working on"
tags: [projects, software, portfolio, systems]
image:
  feature: cover-projects.jpg
---

[This page is currently outdated. Please check my LinkedIn profile instead]

Here you can find more about some of the projects I've developed during my career:

- [Aulas Coletivas (Group classes)](#aulascoletivas)
- [Mobile Micro Training](#mobilemicrotraining)
- [Micro Marketing](#micromarketing)
- [Team Nogueira - Purchasing System](#tnpurchasing)
- [Arquivo Contemporâneo](#arquivocontemporaneo)
- ["Gym in the clouds" - Micropag (Payment Module)](#annmicropag)
- ["Gym in the clouds" - Statistics Module](#annstatistics)
- [Configuration and migration to Amazon EC2 and RDS](#migrationaws)
- [fbuploadtopage - Script to upload images to a Facebook page](#fbuploadtopage)
- ["Gym in the clouds" - Marketing Module](#annmarketing)
- ["Gym in the clouds" - Manager Module](#annmanager)
- ["Gym in the clouds" - Calendar Module](#anncalendar)
- [NBNews](#nbnews)
- [Rio Criativo](#riocriativo)
- [Myth](#myth)
- [Frank Shipbrokers](#fsb)

* * *

<a name="aulascoletivas"></a>Aulas Coletivas (Group classes)
-------------------------------

These mobile apps and web system were developed for the gyms to control their schedules and see which clients were present and the qualifications for each trainer. 

There's a mobile app for the gym clients, so they can book their activities and qualify them when they are over, and there's a mobile app for the trainers, where they can see who's booked at a certain activity, include more people, mark them as present and take photos in order to prove the presences they marked for that activity. 

The two mobile apps were developed by me using the **Ionic Framework**, so they can be converted into Android, iPhone or Windows Phone apps. The Ionic Framework uses **HTML5**, **CSS**, **AngularJS** and **Cordova**. Both apps have QRCode Reader and camera integration. 

I've also developed an API in **.NET** and hosted it on **Amazon EC2**, which connects to **MySQL** databases on **Amazon RDS**, for the mobile apps to consume and fetch the data from. There's also a web system developed in **ASP.NET**, which I haven't developed all by myself, but participated with my team. 

This web system enables the gym to control the activities, schedules, trainers and clients, as well as see reports about everything related to the mobile apps. I've developed an API in .NET to receive images, resize them and store them on **Amazon S3** for the web system and mobile apps to use for uploading activity and profile pictures. I've integrated both mobile apps with an API I've developed in .NET to manage and display some ads (using Amazon EC2, RDS and S3), as well as gather information about their views, clicks, etc.<br /><br />
[![Trainer - Booked people](/images/projects/thumbs/aulas_coletivas_00_tn.png)](/images/projects/aulas_coletivas_00.png)[![Book](/images/projects/thumbs/aulas_coletivas_01_tn.png)](/images/projects/aulas_coletivas_01.png)[![Qualify the activity](/images/projects/thumbs/aulas_coletivas_02_tn.png)](/images/projects/aulas_coletivas_02.png)

* * *

<a name="mobilemicrotraining"></a>Mobile Micro Training
---------------------

This was a software developed for the clients of the gyms to view their trainings on their mobile phones. 

Developed in **jQuery** with **Bootstrap**, consuming the Micro Training web API via **Ajax**, with no server-side language, so it can be ported to **PhoneGap** and compiled to run on both Android and iPhone, as well as other platforms. 

Ps: The names in the screenshots have been blurred for privacy. The software was developed for [Micro University](http://microuniversity.com.br)<br /><br />
[![mobiletraining_00](/images/projects/thumbs/mobiletraining_00_tn.png)](/images/projects/mobiletraining_00.png)[![mobiletraining_01](/images/projects/thumbs/mobiletraining_01_tn.png)](/images/projects/mobiletraining_01.png)

* * *

<a name="micromarketing"></a>Micro Marketing
---------------

This desktop software was developed to integrate with the [Micro Fitness](http://sistemas.microuniversity.com.br/portal/interna.php?conteudo_pagina=13) software and fetch all client data from it through filters set by the user. 

The users can also schedule events for their clients and send e-mails for selected groups of clients who were previously filtered, using templates they have created or just simple text. 

They can also schedule e-mail templates to be automatically sent, send **SMS** and post to the **Facebook** timeline of their clients. The system was integrated with **Amazon SES**, **MailChimp** and **SMTP**, so the user user can choose a service to send the e-mails for his clients. 

It's also integrated with **Amazon S3** and **Dropbox**, so the user can choose a service to store the templates. Developed using **VB.NET** with **MySQL** distributed databases and **SQL Server**. 

The software was developed for [Micro University](http://microuniversity.com.br)<br /><br />
[![micromarketing](/images/projects/thumbs/micromarketing_tn.png)](/images/projects/micromarketing.png)

* * *

<a name="tnpurchasing"></a>Team Nogueira - Purchasing System
---------------------------------

This desktop software was developed for the Team Nogueira franchises to request and purchase their products and for the franchisor to add new available products and control the requests from the franchises. 

Developed using **VB.NET** with **MySQL** for [Micro University](http://microuniversity.com.br)<br /><br />[![tnpedidos](/images/projects/thumbs/tnpedidos_tn.png)](/images/projects/tnpedidos.png)

* * *

<a name="arquivocontemporaneo"></a>Arquivo Contemporâneo
---------------------

This was the website for this furniture design store called Arquivo Contemporâneo (Contemporary file). 

There's a **CMS** that has been specifically developed for them to be able to change the content of the whole website. 

I had to develop new features for it, new areas and fix some bugs as well. Developed using **PHP** with **CodeIgniter** framework and **MySQL**, and **jQuery** with **Ajax**. The system was developed for [NETbureau](http://netbureau.com.br)<br /><br />[![arquivoconte_00](/images/projects/thumbs/arquivoconte_00_tn.png)](/images/projects/arquivoconte_00.png)[![arquivoconte_01](/images/projects/thumbs/arquivoconte_01_tn.png)](/images/projects/arquivoconte_01.png)

* * *

<a name="annmicropag"></a>Academia nas Nuvens (Gym in the clouds) - Micropag (Payment Module)
-------------------------------------------------------------------

This was the payment and credits module for the Gym in the clouds system. It was developed so that the clients from Micro University could buy credits in order to use them on other Gym in the clouds modules. 

I've participated in it's development by adding new features and fixing some bugs. 

Developed using **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. 

Ps: The name in the screenshot has been blurred for privacy 

The system was developed for [Micro University](http://microuniversity.com.br)<br /><br />[![micropag_01](/images/projects/thumbs/micropag_01_tn.png)](/images/projects/micropag_01.png)

* * *

<a name="annstatistics"></a>Academia nas Nuvens (Gym in the clouds) - Statistics Module
-----------------------------------------------------------

This was the Statistics module for the Gym in the clouds system. It was developed so that the clients from Micro University could have several kinds of reports about their gyms and their clients. 

I've participated in it's development by adding new features and fixing some bugs. 

Developed using **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. 

The system was developed for [Micro University](http://microuniversity.com.br)<br /><br />[![estatistica_00](/images/projects/thumbs/estatistica_00_tn.png)](/images/projects/estatistica_00.png)[![estatistica_01](/images/projects/thumbs/estatistica_01_tn.png)](/images/projects/estatistica_01.png)

* * *

<a name="migrationaws"></a>Configuration and migration of all the web systems to Amazon EC2 and RDS
------------------------------------------------------------------------

![LAMP](/images/projects/lamp-stack.jpg)![AWS](/images/projects/aws_logo.jpg)<br /><br />I've configured **Amazon EC2** instances with **Debian GNU/Linux**, **Apache** and **PHP** for the migration of the Academia nas Nuvens systems. 

I've also configured **Amazon RDS** instances with **MySQL** using InnoDB with Multi-AZ replication for greater availability. Also have done Apache and Linux Hardening.

* * *

<a name="fbuploadtopage"></a>fbuploadtopage - Script to upload images to a Facebook page
-----------------------------------------------------------

This php software was developed in order to be integrated with an augmented reality software. 

The augmented reality software would save photos of customers in a directory, the **php** app would read this directory and upload the photos to the company's facebook page, using the **Facebook API**. 

Developed using **PHP**, **PDO** and **SQLite** for [NETbureau](http://netbureau.com.br)

* * *

<a name="annmarketing"></a>Academia nas Nuvens (Gym in the clouds) - Marketing Module
----------------------------------------------------------

This was the Marketing module for the Gym in the clouds system. It was developed so that the clients from Micro University could use it to send **Newsletters** integrated with other system from the company in a transparent way for them. 

It allows the creation of models, it uses filters populated from other systems, automatic sending of e-mails (specific dates or days of the week), click report, received e-mails report, opt-out and views. 

Developed using **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. It also uses **Webservices** to integrate with external systems to send the e-mails via **SOAP** and it has a **REST API** to be integrated with the company's systems. 

The interface was created with **Twitter Bootstrap**. The system was developed for [Micro University](http://microuniversity.com.br)<br /><br />[![Módulo Marketing - Academia nas Nuvens](/images/projects/thumbs/crm_tn.jpg "Módulo Marketing - Academia nas Nuvens")](/images/projects/crm.jpg)

* * *

<a name="annmanager"></a>Academia nas Nuvens (Gym in the clouds) - Manager Module
--------------------------------------------------------

This was the Manager module for the Gym in the clouds system. Gym owners could manage their clients, plans and have some reports. 

I've participated in it's development by adding new features and fixing some bugs. 

Developed using **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. The system was developed for [Micro University](http://microuniversity.com.br)<br /><br />[![gestor_01](/images/projects/thumbs/gestor_01_tn.png)](/images/projects/gestor_01.png)[![gestor_00](/images/projects/thumbs/gestor_00_tn.png)](/images/projects/gestor_00.png)

* * *

<a name="anncalendar"></a>Academia nas Nuvens (Gym in the clouds) - Calendar Module
---------------------------------------------------------

This was the calendar module for the "Gym in the clouds" system. It was developed so that the gyms and pilates studios could control the schedules of each one of their clients in a daily, weekly or monthly view, have attendance reports, activities and their schedules, places where they're going to happen and how many people are scheduled for them. 

They can also control their professionals and the amount of money they will receive, and have some kinds of reports for all system activity. 

Developed using **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. The system was developed for [Micro University](http://microuniversity.com.br)<br /><br />[![Academia nas Nuvens - Módulo Agenda - Cliente](/images/projects/thumbs/academianasnuvens_agenda_01_tn.jpg "Academia nas Nuvens - Módulo Agenda - Cliente")](/images/projects/academianasnuvens_agenda_01.jpg)[![Gym in the clouds - Calendar and scheduling module](/images/projects/thumbs/academianasnuvens_agenda_02_tn.jpg "Gym in the clouds - Calendar and scheduling module")](/images/projects/academianasnuvens_agenda_02.jpg)

* * *

<a name="nbnews"></a>NBNews
------

System for **Newsletter management** and to send Newsletters to mail lists of registered clients, separated by categories, with reports such as: clicks, views, how many people have unsubscribed from the mail list, etc. 

It has an intuitive interface, so the client can create the Newsletter and keep track of the statistics through a complete report. It was developed in **PHP** with **CodeIgniter** framework and **MySQL**, using distributed databases, and **jQuery** with **Ajax**. 

System core developed in **C**, running on **FreeBSD**, using the **sendmail** software. Database backup **ShellScript** [developed by me](https://github.com/jonathas/JonMyBackup), run by cron every week. Modelling and analysis done on MySQL Workbench and ArgoUML, documentation in PHPDoc and user guide in PDF. 

Layout and HTML/CSS by [Erik Dana](http://erikdana.com/) 

The system was developed for [NETbureau](http://netbureau.com.br)<br /><br />
[![NBNews](/images/projects/thumbs/nbnews_tn.jpg "NBNews")](/images/projects/nbnews.jpg)

* * *

<a name="riocriativo"></a>Rio Criativo
------------

System for registration of users interested in subscribing for a business incubator, also management of all the process where the evaluators had to choose the finalists. 

Developed in **PHP** with **CodeIgniter** framework, **MySQL** and **jQuery**. Modelling and analysis done on MySQL Workbench and ArgoUML, documentation in PHPDoc, presentation about the system at [PUC-Rio](http://www.puc-rio.br/). 

HTML/CSS by [Kevin](http://kevinchevallier.com) 

The system was developed for [NETbureau](http://netbureau.com.br)<br /><br />
[![riocriativo](/images/projects/thumbs/riocriativo_tn.jpg "riocriativo")](/images/projects/riocriativo.jpg)

<iframe width="560" height="315" src="https://www.youtube.com/embed/Bagbafpa0OU" frameborder="0"></iframe>

* * *

<a name="myth"></a>Myth
----

Website created in **HTML** and **CSS** with lots of **jQuery** for the effects and transitions. 

Development of a content management system for the "Clipping" and "Lojas" (Stores) areas. 

The system was developed in **OO PHP** with **CodeIgniter** framework and **MySQL**. Modelling and analysis done on MySQL Workbench and ArgoUML, documentation in PHPDoc. 

Layout by [Erik Dana](http://erikdana.com) and HTML/CSS by the two of us.  
Created and Developed for [NETbureau](http://netbureau.com.br)<br /><br />[![myth2](/images/projects/thumbs/myth2_tn.jpg "myth2")](/images/projects/myth2.jpg)[![myth1](/images/projects/thumbs/myth1_tn.jpg "myth1")](/images/projects/myth1.jpg)

* * *

<a name="fsb"></a>Frank Shipbrokers
-----------------

System for management and search of the company's ships and clients Developed in **PHP** with **MySQL** and **jQuery**, using some classes found in [PHPClasses.org](http://www.phpclasses.org). 

An already existent database from 1987 was exported to **MySQL**, I've installed and configured a local server with **Debian GNU/Linux** with a data synchronization system developed in **PHP** and **ShellScript** with **rsync** and ssh. ntp was used to keep the clock in the local system always correct for the sync. 

Database backup ShellScript run by cron every week. Modelling and analysis done on PowerArchitect and ArgoUML, documentation in PHPDoc and user guide in PDF. 

Layout and HTML/CSS by [Kevin](http://kevinchevallier.com) 

The system was developed for [NETbureau](http://netbureau.com.br)<br /><br />
[![FrankShipBrokers](/images/projects/thumbs/fsb1_tn.jpg "FrankShipBrokers")](/images/projects/fsb1.jpg)[![FrankShipBrokers](/images/projects/thumbs/fsb2_tn.jpg "FrankShipBrokers")](/images/projects/fsb2.jpg)
