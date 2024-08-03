# TADPole
*This page is a work-in-progess.*

## Abstract
Temporal models that use timestamped data from electronic health record (EHR) systems could enhance patient outcomes and reduce costs in point-of-care (POC) settings, especially for complex prediction tasks. However, a major obstacle to their widespread use is data drifts, which are changes in how input data relates to target outcomes. Currently, no studies have systematically categorized these data drifts for temporal EHR models or identified the key questions needed to address their impact. This study aims to create a conceptual framework, TADPole, for characterizing data drifts specifically for temporal models in POC environments.

## Example Usage
We outline an example of how to use the framework to guide development of model implementation in a clinical setting. In this example, we focus on in-hospital mortality as the outcome of interest.

 | **PROCESSES** | | | |
|---|---|---|---|
| **Component** | **Definition** | **Questions to Consider** | **Response for Example** |
| Prediction task selection | Selection of outcome and time windows of interest. | 1) Who is the population of interest for the prediction model?<br>2) Which clinical experts will be consulted for development of the prediction task?<br>3) What prediction task and time windows of interest best support the outcome?<br>4) How often will the prediction task be updated?  | 1) ICU patients at a large, urban hospital.<br>2) Critical care physicians, especially those who specialize in pulmonary care to capture knowledge around ventilation and respiratory therapies.<br>3) We will use the first 24 hours of measurements (aggregated hourly) to predict in-hospital mortality within the remainder of the patient stay. Therefore, the observation window is 24 hours, the gap window is 0 hours, and the time-at-risk window is variable depending on the patient stay.<br>4) The prediction task will not be updated unless model performance is severely affected. |
| Feature selection | Selection of input features for modeling. | 1) Who is responsible for feature selection and development?<br>2) Which clinical experts should be consulted for development of the features?<br>3) What features are related to the outcome of interest? <br>4) What biases related to these features should be considered, such as in the way the data is collected or sampled? <br>5) What features are easily, frequently, and accurately collected? <br>6) Are there any features identified that are not easily, frequently, and accurately collected?<br>7) When should the feature set be reviewed and updated? <br>8) When should the features be assessed for drift?  | 1) The model developers and clinical experts will be responsible for feature selection using expert knowledge and feature selection techniques.<br>2) The same clinical experts used for the prediction task selection.<br>3) The features included for further selection will be vital signs, laboratory measurements, ventilation-related variables, and medications.<br>4) We will have to mitigate sampling bias by ensuring that all ICU patient records are available (for example, there are no issues with getting records from a specific department that would affect recording of the features).<br>5) Most of these features are easily collected since they are common EHR features.<br>6) We will have to verify that more potentially subjective variables, like those relating to use of different respiratory therapies, are accurately collected in most cases.<br>7) We will review the feature set either if we notice considerable performance decline or data drifts or if there are changes in the way the features are encoded.<br>8) The features will be assessed for drift starting when the model is implemented and enough data is available for the drift detection methods. |
| Drift detection method selection | Selection of drift detection method(s) that are best suited for the prediction task, drifts of interest, and implementation setting.  | 1) What are any expected sources of population drift?<br>2) Who will be responsible for (a) development and testing of the drift detection methods, (b) tuning the data drift methods once in production, and (c) deciding when detected drift warrants any changes to production, such as model updates?<br>3) What types of drifts are targets for detection and monitoring?<br>4) What methods are applicable to both the drifts of interest and temporal modeling?<br>5) What method provides the most information about the data drifts OR most accurately detects data drifts for the prediction task?<br>6) How often will drifts be monitored?<br>7) What will be considered a significant drift that warrants further action?<br>8) When will updates be expected if significant drifts are detected? | 1) Since we are only implementing the model at a single site, we are not expecting population drift unless the model expands to multiple sites, we see large demographic shifts in the population being served by the hospital, or there is a unique, large-scale event (such as the  COVID-19 pandemic).<br>2) Initial development of the drift methods will be performed by data scientists or related professionals. Production tuning will also be performed by a data scientist hired to monitor the model systems but the overall decision to change the processes will lie with an advisory committee composed of clinical professionals, data scientists and architects, and information technology professionals.<br>3) We will focus most on drifts affected P(X) and P(Y\|X).<br>4) We will test the different methods outlined for temporal models and determine which method performs best with the identified prediction task and training data.<br>5) We will not be sure about this until we do testing.<br>6) The drift methods will perform calculations weekly and provide a report. The timeframe may be adjusted later depending on how frequently false alarms are triggered.<br>7) A significant drift will depend upon the method selected.<br>8) Updates will be prioritized depending upon the severity of the drift but model updates should be expected within several weeks following detection of severe drift. |
| **WORK SYSTEMS** |  |  |  |
| **Component** | **Definition** | **Questions to Consider** | **Response for Example** |
| Location selection | Selected of where the model will be implemented. | 1) Who will make the final decision regarding the implementation setting and the affected population?<br>2) Who will be responsible for drift tuning and monitoring in each location?<br>3) How will drift detection results be communicated to leaders between each location?<br>4) What POC settings will the model be implemented in?<br>5) What are differences in clinical workflow and/or data collection between locations that would affect the underlying data?<br>7) When will reassessment of location selection occur? | 1) Final decisions will be handled by the advisory committee.<br>2) The data scientist or similar professional will be responsible for tuning and monitoring. The same person may be responsible for multiple locations.<br>3) The advisory committee will include individuals from each added location to ensure communication between locations.<br>4) The model will remain in ICU locations only.<br>5) Workflows at each location will have to be investigated to determine how data collection differs at each location.<br>6) Reassessment will occur annually. |
| Timeframe selection | Selection of the drift detection timeframes. | 1) Who is responsible for evaluating the effect of timeframe selection on accuracy of the drift detection methods?<br>2) What is the optimal timeframe for drift detection based on the prediction task?<br>3) Are there any seasonal, cyclical, or repetitive patterns to account for?<br>4) When will reassessment of the timeframe selection occur?  | 1) The data scientists or similar professionals will be responsible for evaluating the effect of the timeframe on drift detection methods.<br>2) The optimal timeframe will have to be tested but initial testing will start with a day and then test larger timeframes.<br>3) We are not aware of any patterns to account for but we may look at infectious disease patterns to ensure these do not have cyclical patterns that may affect drift detection results.<br>4) Reassessment will occur annually. |
| Change detection | Identification of processes to monitor changes in internal processes and model performance. | 1) What are the subpopulations of interest?<br>2) Who will be responsible for overseeing updates to the production models?<br>3) What monitoring mechanisms will be used to ensure changes to labels and features are flagged?<br>4) What parts of the update procedures will be automated? | 1) We are interested in understanding in-hospital mortality statistics for different racial minority groups so we will include monitoring for these groups specifically.<br>2) The advisory committee will be responsible for overseeing updates and ensuring that proper steps are taken.<br>3) Different alert systems will be built to alert when new codes are added, a different coding system is implemented, updates are made to the output of continuous monitoring machines, and other operational changes that affect the selected features.<br>4) The automated parts will be the alerts for changes to the system, alerts for drift detection, and any additional model updating processs that can be implemented without human intervention. |

