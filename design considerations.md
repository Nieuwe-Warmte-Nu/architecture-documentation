# Design considerations Frontend+OMOTES
1.	Gebruiker moet met verschillende modellen (schematisaties) kunnen werken
2.	Gebruiker moet meerdere simulaties tegelijk kunnen starten
3.	Verwachting qua data, 1 jaar met uur tijdstap of 5 jaar met dag tijdstap. Model 1000 componenten en ongeveer 10 uitvoer tijdseries per component -> 90 miljoen data punten
4.	Gebruikers hebben alleen toegang tot eigen modellen en resultaten
5.	Gebruikers kunnen andere toegang geven tot hun modellen en resultaten
6.	Om te voorkomen dat een gebruiker met een enkele berekening heel lang moet wachten op een andere gebruiker die veel modellen draait en het rekencluster op capaciteit is, moet het mogelijk zijn een QoS strategy uit te voeren op de wachtrij van berekeningen.
7.	Gebruikers moeten van 1 project de resultaten van meerdere variaties makkelijk met elkaar kunnen vergelijken in de UI. Denk bijv. aan de time series van een enkele entiteit of KPI met meerdere variaties in 1 grafiek zichtbaar moet zijn en er variaties uit+aan gezet moeten kunnen worden.
8.	Backup strategie voor Design Toolkit moet ook de lopende berekeningen / jobs kunnen herstarten als het cluster faalt.
9.	Maintain session after closing browser
10.	Users must be required to authenticate.
11. Users must be given authorization before accessing calculations & ESDL's from others.
12. Results should be available until frontend and/or the user decides otherwise.
13. Visualisations should include timeseries & combining time series.
14.	Completely SAAS
15.	Must have a support helpdesk.
16.	The architecture must be scalable to hundreds of parallel users.
17.	Core of architecture (models, orchestrations) must be reusable open source.
18. As a user I would like an estimate of how long a computation will run for while configuring it. This will show the consequence in execution time when I am trying various variations.
19. As a user I would like sensible defaults when I configure a computation dynamically based on my ESDL model.
20. As a sysadmin, I need to be able to update the key-figures database based on a new version of static key-figures
21. As the system, I need to be able to retrieve dynamic key-figures based on some endpoints.
22. As an external system, I want to be able to interact with InfluxDB directly with only authorized data. (Perhaps split calculations into seperate db's and allow each user to only use specific db's?)

# Design considerations Frontend
- Integration of other external systems which provide more data should be handled by Frontend. Examples: Heat Profile Generator, Energy Data Repository
- Keeps list of historical jobs.
- 

# Design considerations OMOTES
- No notion of users, authentication, authorization or external systems.
- Guarantees a workflow is either calculated (eventually) or an error is returned.
- Should we be HA, disaster recovery, maximum scalable?
	- Request Stephan & Wouter to think about how many users concurrently should have access in TPG business estimations. (Guestimation: 50?)
    - Monitoring.
    - Logging centrally.
    - Single node is okay
    - Backups of data are necessary (but consistent across multiple databases).
    - HA is not necessary.
    - Health status is necessary.
- Assume fixed worker count or infinite horizontal scalable system.
- Show progress & send an email when done.
- 

## Workers
- Workers can be added and removed dynamically. This allows for easier updates.
- Workflow types are defined by the current workers in the cluster and not apriory. This allows new workflow types to be added 
	- How to handle a workflow that uses multiple models in sequence?
- Should be horizontally scalable but we assume that hardware is unlimited

# Vocabulary
- Workflow: a type of calculation which simulates, optimizes or calculates one or more KPI's. Not the actual running calculation but the description of how to the calculation should happen including code, orchestration and others.
- Job: A new workflow request. The actual calculation.
- Worker: A computer program / service which can perform jobs.
