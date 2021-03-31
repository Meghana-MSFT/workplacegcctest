[[/Images/architecture_overview.PNG|Workplace Awards architecture diagram]]

The **Workplace Awards** app has the following main components: 

**Bot**

The bot is built using the [Bot Framework SDK v4 for .NET](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0) and [ASP.NET Core 2.1](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1). The bot has a conversational interface in team scope for users. 

Workplace Awards Bot provides all end users (internal users seeking help from a central team) an easy interface (bot) to:

* Configure the app captain from list of members in team.

* Nominate team members once the reward cycle is active.

* Endorse other award nominations. You can endorse an award nomination only once.

**Tab**

Manage Rewards tab will show all award nominations in current cycle. App captain will use this tab to publish results for the active reward cycle. Other users will be able to see a historical view of winners from previously published reward cycles.

It will have three action buttons:

-   Publish Results: App captain will select winners from each award category and click on Publish results which will publish the list of winners in the channel.
    
-   Manage Reward Cycle: Manage new cycle will open a task module. The Task module will have sub tabs:  *Manage Awards* & *Set Reward Cycle*. The app captain will be able to perform CRUD operations on Award categories and schedule a one-time or recurring reward cycle.

- Configure app captain: Team member performing the role of the app captain can select a different team member as the app captain.
  

**Messaging Extension**

A messaging extension with [query commands](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/messaging-extensions/search-extensions), which the team can use to search for nominations of current reward cycle in the team. It also implements messaging action that user can use to nominate other team members for the reward.