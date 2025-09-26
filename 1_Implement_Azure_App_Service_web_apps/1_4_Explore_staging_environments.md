# Explore staging environments

When you deploy your web app, web app on Linux, mobile back end, or API app to Azure App Service, you can use a separate deployment slot instead of the default production slot. This approach is available if you run in the Standard, Premium, or Isolated App Service plan tier. Deployment slots are live apps with their own host names. App content and configuration elements can be swapped between two deployment slots, including the production slot.

Deploying your application to a nonproduction slot has the following benefits:

- Validate app changes in a staging deployment slot before swapping it with the production slot.
- Deploying an app to a slot first and then swapping it into production ensures that all instances of the slot are warmed up before being swapped into production. This eliminates downtime when you deploy your app. The traffic redirection is seamless, and no requests are dropped because of swap operations. You can automate this entire workflow by configuring auto swap when pre-swap validation isn't needed.
- After a swap, the previous production app is located in the staging slot. If the changes swapped into the production slot aren't as you expect, you can perform the same swap immediately to get your "last known good site" back.

Each App Service plan tier supports a different number of deployment slots. There's no extra charge for using deployment slots. To find out the number of slots your app's tier supports, visit App Service limits.

To scale your app to a different tier, make sure that the target tier supports the number of slots your app already uses. For example, if your app has more than five slots, you can't scale it down to the Standard tier, because the Standard tier supports only five deployment slots.

When you create a new deployment slot the new slot has no content, even if you clone the settings from a different slot. You can deploy to the slot from a different repository branch or a different repository.

The slot's URL has the format `http://sitename-slotname.azurewebsites.net`. To keep the URL length within necessary domain name system limits, the site name is truncated at 40 characters. The combined site name and slot name must be fewer than 59 characters.

## Examine slot swapping

When you swap two slots, App Service completes the following process to ensure that the target slot doesn't experience downtime:

1. Apply the following settings from the target slot (for example, the production slot) to all instances of the source slot:

   1.1. Slot-specific app settings and connection strings, if applicable.
   1.2. Continuous deployment settings, if enabled.
   1.3. App Service authentication settings, if enabled.

2. Wait for every instance in the source slot to complete its restart. If any instance fails to restart, the swap operation reverts all changes to the source slot and stops the operation.

3. If local cache is enabled, trigger local cache initialization by making an HTTP request to the application root ("/") on each instance of the source slot. Wait until each instance returns any HTTP response. Local cache initialization causes another restart on each instance.

4. If auto swap is enabled with custom warm-up, trigger Application Initiation by making an HTTP request to the application root ("/") on each instance of the source slot.

5. If all instances on the source slot are warmed up successfully, swap the two slots by switching the routing rules for the two slots. After this step, the target slot (for example, the production slot) has the app previously warmed up in the source slot.

6. Now that the source slot has the pre-swap app previously in the target slot, perform the same operation by applying all settings and restarting the instances.

At any point of the swap operation, all work of initializing the swapped apps happens on the source slot. The target slot remains online while the source slot is being prepared and warmed up, regardless of where the swap succeeds or fails. To swap a staging slot with the production slot, make sure that the production slot is always the target slot. This way, the swap operation doesn't affect your production app.

When you clone configuration from another deployment slot, the cloned configuration is editable. Some configuration elements follow the content across a swap (not slot specific), whereas other configuration elements stay in the same slot after a swap (slot specific). The following table shows the settings that change when you swap slots.

## Swap deployment slots

You can swap deployment slots on your app's Deployment slots page and the Overview page. Before you swap an app from a deployment slot into production, make sure that production is your target slot and that all settings in the source slot are configured exactly as you want to have them in production.

### Manually swapping deployment slots

To swap deployment slots:

1. Go to your app's Deployment slots page and select Swap. The Swap dialog box shows settings in the selected source and target slots that are changed.

2. Select the desired Source and Target slots. Usually, the target is the production slot. Also, select the Source Changes and Target Changed tabs and verify that the configuration changes are expected. When you are finished, you can swap the slots immediately by selecting `Swap`.

3. When you're finished, close the dialog box by selecting `Close`.

### Swap with preview (multi-phase swap)

Before you swap into production as the target slot, validate that the app runs with the swapped settings. The source slot is also warmed up before the swap completion, which is desirable for mission-critical applications.

When you perform a swap with preview, App Service performs the same swap operation but pauses after the first step. You can then verify the result on the staging slot before completing the swap.

If you cancel the swap, App Service reapplies configuration elements to the source slot.

### Configure auto swap

Auto swap streamlines Azure DevOps Services scenarios where you want to deploy your app continuously with zero cold starts and zero downtime for customers of the app.

Note: Auto swap isn't currently supported in web apps on Linux and Web App for Containers.

### Specify custom warm-up

Some apps might require custom warm-up actions before the swap. The `applicationInitialization` configuration element in web.config lets you specify custom initialization actions. The swap operation waits for this custom warm-up to finish before swapping with the target slot. Here's a sample web.config fragment.

### Roll back and monitor a swap

If any errors occur in the target slot (for example, the production slot) after a slot swap, restore the slots to their pre-swap states by swapping the same two slots immediately.

If the swap operation takes a long time to complete, you can get information on the swap operation in the activity log.

1. On your app's resource page in the portal, in the left pane, select `Activity log`.
2. A swap operation appears in the log query as `Swap Web App Slots`. You can expand it and select one of the sub-operations or errors to see the details.

## Exercise - Swap deployment slots in Azure App Service

In this exercise, you deploy a basic HTML+CSS site to Azure App Service with the Azure CLI `az webapp up` command. Next, you update the code and deploy the change to a staging slot. Finally, you swap the slots.

Tasks performed in this exercise:

- Download and deploy the sample app to Azure App Service.
- Create a staging deployment slot.
- Make a change to the sample app and deploy it to the staging slot.
- Swap the staging and default production slots to move the changes to the production slot.

## Route traffic in App Service

By default, all client requests to the app's production URL (`http://<app_name>.azurewebsites.net`) are routed to the production slot. You can route a portion of the traffic to another slot. This feature is useful if you need user feedback for a new update, but you are not ready to release it to production.

### Route production traffic automatically

To route production traffic automatically:

1. Go to your app's resource page and select Deployment slots
2. In the Traffic % column of the slot you want to route to, specify a percentage (between 0 and 100) to represent the amount of total traffic you want to route. Select Save.

After the setting is saved, the specified percentage of clients is randomly routed to the nonproduction slot.

After a client is automatically routed to a specific slot, it's "pinned" to that slot for the life of that client session. On the client browser, you can see which slot your session is pinned to by looking at the `x-ms-routing-name` cookie in your HTTP headers. A request routed to the "staging" slot has the cookie `x-ms-routing-name=staging`. A request routed to the production slot has the cookie `x-ms-routing-name=self`.
