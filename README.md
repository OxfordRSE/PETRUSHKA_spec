# PETRUSHKA shared variable specifications

## System components 

This is a list of the variables that are shared by two or more parts of the system. 
The parts of the system that have distinct data requirements are:
* `Sentry` -- the Sentry API gateway to the OpenClinica database
* `Django` -- the web front-end that faces the clinician and the patient and acts as a go-between for the `Sentry` and `Algorithm` components
* `Algorithm` -- the machine learning algorithm that helps clinicians match patients to drugs by adverse effect profile
  * `OBS` -- Subcomponent of the algorithm
  * `RCT` -- Subcomponent of the algorithm

### Workflow
```puml

participant Algorithm
participant Django
participant Sentry

hnote over Django : Clinican login
hnote over Django : Patient token submitted

Django -> Sentry : Request patient data\n(""{token, clinician}"")
hnote over Sentry : Check clinician access
Sentry -> Django : Send patient data
Django -> Sentry : Send acknolwedgement\n(lock future access)

Django -> Algorithm : ""patient_data""
hnote over Algorithm : Select adverse effect options
Algorithm -> Django : ""[ adverse effect options ]""
hnote over Django : Patient ranks and scores\nadverse effects

Django -> Algorithm : ""patient_data, ae_preferences""
hnote over Algorithm : Identify drug options 
Algorithm -> Django : ""[ drug options ]""
hnote over Django : Patient selects preferred drug\nin consultation with clinician

Django -> Sentry : Send patient choice\n(""{token, clinician, preference}"")

```
![Workflow overview](workflow.png)

### Interfaces

There are two key interfaces -- **Sentry**<-->**Django** and **Django**<-->**Algorithm**.
Data exchanged between **Sentry** and Django will be in **json** format. 
Data exchanged between Django and **Algorithm** will be in **python** format, either **dict** or a list of **dict**s -- keys will be the variable 
name and values the variable value.
The details of these requests are specified below the Variables list.

## Variables

**Version**: `0.1.1`

| Variable                                | Type        | Used by           | Description                                                                                                                 |
|-----------------------------------------|-------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------|
| **token**                               | string      | Sentry, Django    | Patient unique identifier. In the form `[PS][0-9]{4}`                                                                       | 
| clinician                               | string      | Sentry, Django    | Username of the clinician to whom this patient is assigned                                                                  |
| male_at_birth                           | bool        | All               | Whether patient was assigned male at birth                                                                                  |
| age                                     | int         | All               | Patient age in years                                                                                                        |
| first_episode                           | bool        | Sentry, OBS       | Whether this is the patient's first episode of depression                                                                   |
| fluoxetine_history                      | bool        | Sentry, OBS       | Whether this patient has been prescribed fluoxetine before                                                                  |
| ssri_history                            | bool        | Sentry, OBS       | Whether this patient has been prescribed SSRIs before                                                                       |
| antidepressant_history                  | bool        | Sentry, OBS       | Whether this patient has been prescribed antidepressants before                                                             |
| bmi                                     | double      | Sentry, OBS       | Patient's body mass index (kg/m^2)                                                                                          |
| smoking_history                         | bool        | Sentry, OBS       | Whether patient has ever been a smoker                                                                                      |
| phq_9                                   | int         | Sentry, OBS       | Patient Health Questionnaire (PHQ-9) baseline depression severity score, 0-27                                               |
| psychotherapy_history                   | bool        | Sentry, OBS       | Whether this patient has previously had psychotherapy                                                                       |
| secondary_care_history                  | bool        | Sentry, OBS       | Whether this patient has previously been referred to secondary care                                                         |
| childhood_maltreatment_history          | bool        | Sentry, OBS       | Whether this patient has a history of child maltreatment                                                                    |
| coronary_heart_disease_comorbidity      | bool        | Sentry, OBS       | Whether this patient has comorbid coronary heart disease                                                                    |
| stroke_tia_comorbidity                  | bool        | Sentry, OBS       | Whether this patient has comorbid stroke or transient ischaemic attack                                                      |
| diabetes_comorbidity                    | bool        | Sentry, OBS       | Whether this patient has comorbid diabetes                                                                                  |
| epilepsy_seizure_comorbidity            | bool        | Sentry, OBS       | Whether this patient has comorbid epilepsy or seizures                                                                      |
| hypothyroidism_comorbidity              | bool        | Sentry, OBS       | Whether this patient has comorbid hypothyroidism                                                                            |
| chronic_inflamatory_disease_comorbidity | bool        | Sentry, OBS       | Whether this patient has comorbid chronic inflamatory disease                                                               |
| anxiety_ocd_comorbidity                 | bool        | Sentry, OBS       | Whether this patient has comorbid anxiety or Obsessive-compulsive disorder                                                  |
| migrane_comorbidity                     | bool        | Sentry, OBS       | Whether this patient has comorbid migrane                                                                                   |
| antihypertensive_drug_use               | bool        | Sentry, OBS       | Whether this patient is currently using antihypertensive drugs                                                              |
| asprin_use                              | bool        | Sentry, OBS       | Whether this patient is currently using asprin                                                                              |
| statins_use                             | bool        | Sentry, OBS       | Whether this patient is currently using statins                                                                             |
| non_steroidal_anti_inflamatory_drug_use | bool        | Sentry, OBS       | Whether this patient is currently using non-steroidal anti-inflamatory drugs                                                |
| anticoagulants_use                      | bool        | Sentry, OBS       | Whether this patient is currently using anti-coagulant drugs                                                                |
| anticonvulsant_use                      | bool        | Sentry, OBS       | Whether this patient is currently using anticonvulsant                                                                      |
| hypnotics_anxiolytics_use               | bool        | Sentry, OBS       | Whether this patient is currently using hypnotics or anxiolytics                                                            |
| bisphosphonates_use                     | bool        | Sentry, OBS       | Whether this patient is currently using bisphosphonates                                                                     |
| oral_contraceptive_hrt_use              | bool        | Sentry, OBS       | Whether this patient is currently using oral contraceptives or hormone replacement therapy                                  |
| ethnicity                               | bool        | Sentry, OBS       | Ethnicity (0 white/Caucasian, 1 African/Caribbean, 2 Asian, 3 Other), one-shot encoding                                     |
| townsend_quintile                       | bool        | Sentry, OBS       | Townsend quintile (from 1-least deprived to 5-most deprived), one-shot encoding                                             |
| hamd_3                                  | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 3 (0-4)                                                      |
| hamd_4                                  | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 4 (0-2)                                                      |
| hamd_6                                  | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 6 (0-2)                                                      |
| hamd_10                                 | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 10 (0-4)                                                     |
| hamd_11                                 | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 11 (0-4)                                                     |
| hamd_13                                 | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 13 (0-2)                                                     |
| hamd_17                                 | int         | Sentry, RCT       | Patient's score on Hamilton Depression Rating Scale - Question 17 (0-2)                                                     |
| hamd                                    | int         | Sentry, RCT       | Patient's total overall score on Hamilton Depression Rating Scale (0-52)                                                    |
| ae_probs                                | Array<Dict> | Django, Algorithm | Probabilities of adverse effects. Each probability represents an intersection between a drug and an adverse effect          |
| **preference**                          | Array<Dict> | Django, Algorithm | Patient's preferences for adverse effects. Each adverse effect is a dictionary with keys 'name', 'rank', and 'score'        |
| drug_options                            | Array<Dict> | Django, Algorithm | List of drug options to present to the patient. Each drug is presented as a dictionary with keys and values specified below |
| **drug_chosen**                         | string      | Django, Sentry    | Drug chosen by patient in consultation with clinician                                                                       |
 
**bold** names indicate variables that come from _users_. We should discuss a validation strategy for these variables -- who will validate them for security and how.
 
## Interchange formats

There are 8 interactions between components. 
The details for each of these 8 requests are as follows:

### _Django_ Request patient data _from Sentry_

```json5
{
  "authorisation": "string",  // Access token
  "action": "ptdata",         // Requested operation
  "clinician": "string",      // Clinician identifier
  "token": "string"           // Patient identifier matching RegExpr [PS][0-9]{4}
}
```

### _Sentry_ Send patient data _to Django_

```json5
{
  "token": "string",            // Patient identifier
  "age": "int",                 // Patient age
  "male_at_birth": "boolean",   // Whether patient was assigned male at birth
  // ...etc. for patient information keys listed in the Variables section
}
```

### _Django_ Send acknowledgement _to Sentry_

```json5
{
  "authorisation": "string",  // Access token
  "action": "ptdata_ack",     // Requested operation
  "token": "string",          // Patient identifier matching RegExpr [PS][0-9]{4}
  "status": "success"         // Clinician identifier
}
```

### _Django_ Send patient data _to Algorithm_

```python
patient_data = {
  "age": "int",                 # Patient age
  "male_at_birth": "boolean",   # Whether patient was assigned male at birth
  # ...etc. for patient information keys listed in the Variables section
}
petrushka_backend.get_adverse_effect_options(patient_data)
```

### _Algorithm_ Send adverse effect options _to Django_

```python
# List of adverse effect names. Should be length 3, and the order is not important.
adverse_effect_options = ['ae_1_name', 'weight_gain', 'ae_16_name']
```

#### _Algorithm_: Adverse effect details

The Algorithm should also expose a dictionary of adverse effect names and their associated details:

```python
petrushka_backend.adverse_effect_details = {
  'SE_1': {
    'name': 'ae_1_name',
    'probs': (  # key is the adverse effect's name
      0.01,  # Value is a tuple of low,
      0.10   # high probability of occurence
    ),
    'description': 'You probably don\t want this.'
  },    
  'SE_2': {
    'name': 'ae_2_name',
    'probs': (0.05, 0.25),
    'description': 'You probably don\t want this.'
  },
  # ...etc. for all adverse effects
}
```

### _Django_ Send adverse effect preferences _to Algorithm_

```python
preference = [  # list of dictionaries, each dictionary is an adverse effect. Order is arbitrary.
  {  # Dictionary for adverse effect 1
    'name': 'ae_1_name',  # Adverse effect name
    'rank': 5,  # Rank as ranked by the user. Higher values = more preferred.
    'score': 100  # Score as given by the user. Rank 5 will always have score 100. Higher values = more preferred.
  },
  {
    'name': 'ae_2_name',
    'rank': 4,
    'score': 30  # Note that 'score' and 'rank' are not necessarily related: users can give a higher rank a lower score
  },
  {
    'name': 'weight_gain',
    'rank': 3,
    'score': 75
  }
  # ...etc. for the other 2 AEs included as keys in drug_options[0]['ae_probs']
]
petrushka_backend.get_drug_options(patient_data, preference)
```

`patient_data` as defined in [Django Send patient data to Algorithm](#Django Send patient data to Algorithm).

### _Algorithm_ Send drug options _to Django_

```python
drug_options = [
  {  # option 1
    'name': 'examplin',  # The drug's name (this will not be shown to the user yet). Serves as a unique identifier.
    'predictions': {
      'efficacy': 'double',         # How well drug will work for patient
      'acceptability': 'double',    # How acceptable drug will be to patient
      'ae_1_name': 'double',        # Likelihood of adverse_effect_1
      # ...etc. for the other 4 adverse effects
    }
  },
  # ... for options 2 and 3
]
```

### _Django_ Send patient choice _to Sentry_

```json5
{
  "authorisation": "string",  // Access token
  "action": "ptsession",      // Requested operation
  "clinician": "string",      // Clinician identifier
  "token": "string",          // Patient identifier matching RegExpr [PS][0-9]{4}
  "session": {
    "drug_chosen": "string"   // Name of the drug chosen
  }
}
```

## Updates

We all need to agree on this specification and be responsible for ensuring that our own components follow it.
It's entirely possible that the specification will need updating.
When updates are made, whoever makes the updates must notify the maintainers of other components that require updating as a result.
