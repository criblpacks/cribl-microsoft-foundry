# Cribl Azure Foundry EventHub Collector
----
## About this Pack

This pack is built as a complete SOURCE + DESTINATION solution (identified by the IO suffix). Data collection and delivery happen entirely within the pack's context, eliminating the need to connect it to globally defined Sources and Destinations. 

This Pack is designed to collect, process, and output [Microsoft Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry?view=foundry) AI-related data from an Azure EventHub. The data itself is generated via instrumenting your AI Agent/Application with OTEL-compatible data - see the [documentation](https://learn.microsoft.com/en-us/azure/ai-foundry/observability/concepts/trace-agent-concept?view=foundry) for details.

Please carefully review the configuration section as enabling data collection is a multi-step process.

This Pack targets the "new" Microsoft Foundry - it may or may not work with the "classic" version. 

## Deployment

* This pack is configured by default to use the Worker Group's *Default Destination*.
* To use the *Default Destination*: No changes are required. The pack will route the data to the destination currently set as the Default on the Worker Group.
* To use a different Destination: You must update the pack's routes to specify your desired Destination.
* For immediate functionality without requiring Pack route filter expression modifications, every bundled Source within this pack adds a hidden field: `__packsource`. This field allows for seamless routing based on the Pack source.

### Configure Microsoft Foundry Data Collection
Configuring data collection via Azure Monitor is best done by leveraging one of the [Solution Templates](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/ai-template-get-started?view=foundry) provided by Microsoft. These all come pre-configured with the option to generate trace data and will automatically provision the required Azure resources for you, including Application Insights and LogAnalytics. If you are not using a Solution Template, than you must ensure the following resources exist and are configured properly:
- [Tracing](https://learn.microsoft.com/en-us/azure/ai-foundry/observability/how-to/trace-agent-setup?view=foundry) enabled and working.
- Application Insights connected to your Agents and gathering data.
- A LogAnalytics Workspace where the tracing data is sent.

Foundry data flow into Cribl is `agent trace -> app insights -> log analytics -> event hub -> cribl`. Perform the following to get the data into an Event Hub for Cribl to ingest:
- Create (or reuse) an Application and ensure it has the `Azure Event Hubs Data Receiver` permission. See [here](https://learn.microsoft.com/en-us/azure/event-hubs/authenticate-application) for details. 
- Create an Event Hub namespace and an Event Hub. By default, this Pack uses `microsoft-foundry-cribl-integration` for the Event Hub name. You should *not* use an already-in-use Event Hub! 
- Create a [Data Export Rule](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/logs-data-export?tabs=portal#export-destinations) in your Log Analytics space to export the trace data table(s) to the Event Hub you created above. By default, this data is in the `AppDependencies` and `AppEvents` tables. 

### Configure the Event Hub in Cribl
You will need the following to configure the Event Hub in Cribl:
- Tenant ID, Application/Client ID, and a Client Secret.
- Event Hub Namespace value and Event Hub name.
- If you are *not* using OAUTH authentication, you will need the correct Azure Event Hub connection string.

In the Pack, click on the `in_azure_foundry_cribl_integration` Event Hub source and configure the following:
- In General Settings, replace `YOUR_EVENTHUB_NAMESPACE` in the Brokers entry with your Event Hub Namespace and update the Event Hub name entry (if using a name other than `microsoft-foundry-cribl-integration`).
- In Authentication, add your Tenant ID, Client ID, and Client Secret and replace `YOUR_EVENTHUB_NAMESPACE` with your Event Hub Namespace value.
- Perform a Commit/Deploy to enable data collection.

### Configure Output Format

Each data type can be configured to output data in either normalized JSON or Splunk (`_raw` + Splunk fields) format. Enable *only one* format for the `cribl_microsoft_foundry` pipeline.

### Configure your Destination/Update Pack Routes
To ensure proper data routing, you must make a choice: retain the current setting to use the Default Destination defined by your Worker Group, or define a new Destination directly inside this pack and adjust the pack's routes accordingly.

### Commit and Deploy
Once everything is configured, perform a Commit & Deploy to enable data collection.

#### Variables

The Pack has the following variables:
* `microsoft_foundry_default_splunk_index`: Default index for the Splunk output.

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 1.0.0
- Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).