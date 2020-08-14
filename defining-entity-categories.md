# 2.2 Defining entity categories

In this section, we define the medical categories within the scope of these annotation guidelines. Because of the inherent ambiguous nature of language, we need to prepare guidelines for outlining which terms belong in each medical category. The definitions provided herein are an attempt to clarify what is being annotated and how ambiguous scenarios should be handled.

## Category: Anatomical structure

### Definition

A complex part of the human body. Human organs, cells, and systems of the human body fall under this category.

### Examples and annotation rules

1. “Patient is suffering from lung cancer”
  1. “lung” is an anatomical structure.
  1. Other examples of anatomical body parts: chest, coronary artery, hand, sigmoid colon, and so forth.
1. If an entity is related to multiple anatomical references, then break the whole phrase (even if it is an adjoining word), annotate the entities individually, and relate the anatomical references to the entity.
  1. **For example**: “The patient has chest and abdominal pain.” In this case, “pain” should be annotated as a medical condition. The words “abdominal” and “chest” should be annotated as anatomical structures and should be related to the entity “pain”. Although “abdominal” is an adjoining word, if we annotate it as a full entity, then its context with “chest” will be nullified. Therefore, we need to break the adjoining words in case there are  multiple entities linked together.
1. Anything mentioned in the form of range “Head to Toe” or “Cranial nerves 2-10”  should be annotated together as an anatomical structure so that the whole meaning is retained.

## Category: Body function

### Definition

General function carried out by the human body.

### Examples and annotation rules

1. The following keywords depict body functions:
  1. “Bowel sounds”
  1. “Diastolic function”
  1. “Menstrual cycle”
  1. “postmenopausal”
  1. “psych”

### 2.2.2.1  Body function modifier

The status or result of any body function should be annotated as a body function modifier. For example, body function modifiers include words like “increased”, “decreased”, “positive”, or “negative”.

**Example 1**: Diastolic function is normal. “normal” is the body function result and “diastolic function is” the body function.

## Category: Body measurement

### Definition

A normal measurement of the body that is obtained without performing a complex procedure or test. Vital signs are classified as body measurements. Body measurements also capture values obtained using simple instruments such as a thermometer for temperature, a stethoscope for a pulse, and so forth.

### Examples and annotation rules

1. “Pulse: 80. BP: 110/70. Temp: 97.4”
  1. Body measurement: “pulse”, “BP”, “temperature”. Similarly, “80”, “110/70”, “97.4” are the respective values as defined below.
1. A body measurement consists of all vital signs. Measurements should be annotated irrespective of the values mentioned in the document. 
  1. **For example**: “Blood pressure is normal.”
    1. In this case, even though there is no value mentioned with “blood pressure”, we should annotate “blood pressure” as a body measurement.

### 2.2.1 Modifier: Result

#### Definition

The result of any body measurement should be annotated as a measurement result. It might contain words like “increased”, “decreased”, “positive”, or negative. Generally, such words are modifiers that may depict the status of the body measurement.

#### Examples and annotation rules

* "BP is high."
  * “high” is the result of the body measurement “BP”.

### 2.2.2 Modifier: Value

#### Definition

Values of body measurement fall under this category. Such a value cannot stand alone and can only be annotated in relation to a body measurement.

#### Examples and annotation rules

1. “Pulse: 80. BP: 110/70. Respirations: 16. Temp: 97.4”
  1. “80” is the value of body measurement “pulse”
  1. “110/70” is the value of body measurement “BP”
  1. “97.4” is the value of body measurement “Temp"

### 2.2.3 Modifier: Unit

#### Definition

The unit of the body measurement value.  The annotation of “Unit” cannot stand alone and has to be related to body measurement.

#### Examples and annotation rules

1. “BP: 110/70 mmhg”
  1. “mmhg” is the unit, “BP” is the body measurement, and “110/70” is the value.

## Category: Laboratory data

### Definition

A medical procedure that involves testing a sample of blood, urine, or other substance from the body. Laboratory tests can be used for the following:

* Help determine a diagnosis
* Plan treatment
* Check treatment effectiveness
* Monitor a disease over time

### Examples and annotation rules

1. “White blood cell 3.0, haemoglobin 13.9, and lymphocytes 48.”
  1. “white blood cell”, “haemoglobin”, and “lymphocytes” are the elements that are being analyzed using certain procedures and tests. “White blood cells” represent body substances, but since it has a value assigned, it falls under the category of laboratory data.

### 2.2.x.1 Modifier: Lab value

#### Definition

The value of laboratory data. This annotation cannot stand alone and has to be related to laboratory data.

#### Examples and annotation rules

1. “White blood cell 3.0, haemoglobin 13.9, and lymphocytes 48”
  1. “3.0” is the value of the lab data “white blood cell”.

### 2.2.x.2 Modifier: Lab unit

#### Definition

The unit of measurement for the laboratory value.  This value cannot stand alone and has to be related to laboratory data.

#### Examples and annotation rules

1. “White blood cell 3.0, haemoglobin 13.9 g/dl, and lymphocytes 48”
  1. “g/dl” is the unit of the laboratory value.

### 2.2.x.3 Modifier: Lab result

#### Definition

The result of laboratory data. It may contain qualifiers like “increased”, “decreased”, “positive”, or “negative”. Any organism that is a result of culture or any other lab data performed is a lab result.

#### Examples and annotation rules

1. “The patient has low phosphate and magnesium levels”
  1. "Low” should be annotated as a lab result related to both “phosphate” (lab data) and  “magnesium levels" (lab data).
1. “Increased cholesterol level”
  1. “Increased” is the lab result and “cholesterol level” is the laboratory data.
1. “Culture grew E.coli”
  1. “E.coli” is the lab result and “culture” is the laboratory data.

### 1.3.4.4 Additional rules for annotating lab data attributes

* 2.3 The attributes do not stand alone. Lab data should be present to annotate the attributes.
* 2.4 When “range” is mentioned in the text,it should not be annotated as a lab data value.
  * For example: “Albumin 3.7 g/dL (3.5-5.2) 08/25/18 04:07”. The value is “3.7” and should be linked to “Albumin”. “(3.5-5.2)” is the reference range and should not be annotated.
* 2.5 The relationship boundary is one sentence. Do not relate lab data attributes spanning entities beyond one sentence.
* 2.6 If any symbol is present with the value, such as “>” or “<”, then the value and the symbol  should be annotated together.
* 2.7 If the value is written as "elevated from 1.1 to 3.0", then the last captured lab value should be annotated.
* 2.8 There are cases when multiple values are documented with the lab component. In such cases, all the values are marked and linked with the lab component because it is  not possible to determine which value was last derived.
* 2.9 A status of “positive” or “negative”  of any test result should be marked as an attribute because it is an important factor for interpreting the lab data.
* 2.10 In the case of compounded lab data, annotate the entire snippet including the conjunctions. For example, “check afb and fungal cultures” is annotated as the lab value in the example “check afb and fungal cultures on the synovial fluid”.
* 2.11 If there is a count of cells or organisms measured in NCnc (count/volume), only “/volume” will be annotated under the unit. For example, in “CSF RBC 1590 /uL”, “/ul” is the unit and “1590” is the value of the lab data RBC from the CSF sample.
* 2.12 There are cases when the lab component is written with its synonym and another part of the component is written after. For example, “Rubeola (Measles) IgG < 25.0 AU/mL”. In this sentence, “Rubeola” and “Measles” are synonyms, so the entire snippet “Rubeola (Measles) IgG” should be annotated as a single entity.
* 2.13 If a short form of the lab data is present in addition to the full form, then annotate both the short and long form and link values and units to both entity annotations. For example, “Estimated GFR (eGFR) 11 (>=60)”. The “Estimated GFR” snippet should be annotated as lab data and linked to “11” as its value. Similarly, “eGFR” should be annotated as lab data and linked to “11” as its value.
* 2.14 There are instances when the explicit word “positive” is absent, but a “+” sign is present. In that case, “+” is annotated as a modifier for the lab data so that the meaning of the sentence is retained.
* 2.15 The organisms growing out of a lab culture are considered a lab result. For example, in “sputum growing streptococcus pneumonia and a small amount of candida albicans”:
  * “sputum” is annotated as laboratory data
  * “streptococcus pneumonia” is annotated as a lab result
  * “candida albicans” is annotated as a lab result

## Category: Medical condition

### Definition

A condition that alters or interferes with a normal process, state, or activity of the human body. A medical condition is usually characterized by the abnormal functioning of one or more body systems, parts, or organs. Symptoms that are descriptive of a disorder are also included in this category. Note that any anatomical structure related to the medical condition should be annotated as an “anatomical structure”  and  assigned a relationship to the medical condition.

### Examples and annotation rules

1. “Patient was suffering from lung cancer.”
  * “cancer” is to be annotated as a medical condition. “Lung” is an anatomical body site and should be related to the medical condition “cancer”.
1. When annotating the temporal status of a medical condition, always take into account the closer context.
  1. For example: “Medical History: large cell lymphoma- new diagnosis” 
    1. “Large cell lymphoma” should be annotated as a medical condition, and the status should be annotated as “current”.
1. Adjectives that describe damage to body parts should be annotated as medical conditions. For example: 
  1. “He hasn’t broken anything yet”. “broken” should be annotated as a medical condition.
  1. “Musculoskeletal: instability”. “instability” should be annotated as a medical condition.
1. If there are any abbreviations present, annotate the abbreviation as a full entity.
  1. For example: “COPD.”
    1. “COPD” - chronic obstructive pulmonary disease - is a universally accepted abbreviation, so we annotate the full entity as a medical condition.
    1. For example: “The patient has coronary artery disease.”
      1. Although “coronary artery” is an anatomical structure, it would not be annotated as such. This is because “CAD” - coronary artery disease - stands for coronary artery disease so the full entity “coronary artery disease” should be annotated as a medical condition.
1. The mood status and behaviour pattern of a patient should be annotated under medical condition. 
  1. For example: “irritated”, “agitated”, and so forth.
1. Any organism that is mentioned in the document, should not be annotated as a medical condition.
  1. For example: “Urine culture grew E.coli.” In this case, E.coli should not be annotated as a medical condition.
1. If any entity occurs with an anatomical reference as adjectives, then the whole snippet should be considered an annotation.
  1. For example: “The patient has gastrointestinal bleeding and abdominal pain.” “gastrointestinal bleeding” and “abdominal pain” should be annotated as medical conditions because “gastrointestinal” and “abdominal” are adjectives. If the anatomical references are not adjectives, such anatomical references should not be included in the annotation.
1. If the document contains any information about the medical dependency of the patient, the information should not be annotated under medical condition.
  1. For example: Words like “ventilator-dependent” and “hemodialysis-dependent” should not be considered under medical condition.
1. If there is any mention of abuse, withdrawal, or dependency of a psychotropic substance, it should not be annotated as a medical condition.
  1. For example: “The patient is admitted due to alcohol abuse.” In this case, abuse entities should not be annotated under medical condition.
1. If a medical condition is mentioned inside a form and the form content includes certainty assessment, severity, or location, then we annotate the medical conditions and its attributes accordingly.
1. Annotate "cholesterol" as a medical condition if it is mentioned in the document as "Patient's cholesterol is increased.'' 
1. The mood status and behavior pattern of the patient can be considered medical conditions. There are times when the reason for a visit or the diagnosis of the patient are such entities. For example: in “The patient is confused, irritated and agitated”, “confused, irritated and agitated” should be annotated as a medical condition.
1. When body location is mentioned along with the medical condition, do not annotate the location and the medical condition together.
  1. For example: “The patient has left-sided weakness.” In this case, “weakness” should be annotated as a medical condition.
1. The term “pregnancy” should be annotated as a medical condition because it is a temporary medical condition that interferes with the normal functioning of the female body. Therefore, it influences the delivery of care and we should annotate it as a medical condition.
1. Terms like “spontaneous abortion” and “miscarriage” should be annotated as medical conditions.
1. There are instances where signs or symptoms of labor are mentioned. Such terms should also be considered as a medical condition.
  1. For example: “Interval cervical changes.” This is a sign/symptom of labor. Labor is a medical condition and we need to annotate the signs leading to this event.
1. Terms like “contractions” refer to the body function and hence, regardless of the severity, these terms should not be annotated as medical conditions.

### 2.15.x.1 Modifier: Severity

#### Definition

This modifier should be used to annotate the severity of the medical condition.

#### Examples and annotation rules

1. “The patient has chronic leg pain.”
  1. “Pain”: Medical condition
  1. “Chronic”: Severity
  1. “Leg”: Anatomical structure
1. “Benign” should be annotated as a severity.
  1. For example: “Benign essential hypertension”
    1. “Essential hypertension”: Medical condition
    1. “Benign”: Severity
1. When severity is mentioned along with the medical condition, the severity should be annotated separately except if it is a part of an abbreviation.
  1. For example: “The patient has chronic hand pain.  “
    1. “Pain”: Medical condition
    1. “Chronic”: Severity
1. “Chronic kidney disease, stage 5.”
  1. “Stage 5”: Severity
  1. “Chronic kidney disease”: Medical condition as it is the extended version of “CKD” .

## Category: Medical device

### Definition

Any physical or virtual instrument that is used for medical treatment, operation, and various other medical purposes.

### Examples and annotation rules

1. “Nasogastric tube has now been removed”
  1. “nasogastric tube”: Medical device.
1. Other examples are “Greenfield filter”, “IVC filter”, “X-ray gurney”, “duodenoscope”, “catheter”, and so forth.

## Category: Medical procedure

### Definition

Any diagnostic procedure, examination, surgery, or any treatment procedure that is conducted to treat or to diagnose a condition. If any anatomical structure is related with the procedure performed, then the anatomical structure should also be annotated and linked to the procedure.

### Examples and annotation rules

1. “The patient underwent total knee amputation.”
  1. “total knee amputation”: Procedure
1. Other examples of medical procedures are “CT scan”, “sonography”, “surgery”, “Lymphadenectomy”, and so forth.
1. Removal of masses or body parts should be annotated as medical procedures.
  1. “Skin cancers removed” should be annotated as:
    1. “Skin”: Anatomical body site
    1. “Cancers”: Medical condition
    1. “Removed”: Medical procedure
  1. “Mass of abdomen removed” should be annotated as:
    1. “Mass”: Medical condition
    1. “Abdomen”: Anatomical body site
    1. “Removed”: Medical procedure
1. Medical procedures should be annotated depending on the meaning and the context of the underlying text.
  1. For example: “There is drainage of pus and hence drainage was performed.” In this case, the word “drainage” is mentioned twice, but both times in a completely different context. The first occurrence suggests a problematic condition, whereas the second occurrence refers to the procedure conducted. The occurrences of “drainage” should be annotated as medical condition and medical procedure, respectively.
1. Mentions of medical procedures within names of department, team, center, ward should not be annotated as medical procedures.
  1. For example: “The patient presented to the wound care center.” In this case “wound” should not be annotated because it is in a relationship with “center” and does not symbolize a medical condition.
1. All radiological imaging should be annotated under medical procedure.
1. Entities referring to screening and examination should be annotated as medical procedures.
1. If any disease workup is carried out, then it should be annotated together as a medical procedure.
  1. For example: “Endocarditis work-up” should be annotated together as a medical procedure.
1. Medical procedures should be annotated along with their anatomical reference.
  1. For example: “He underwent a chest x-ray.” In this case, “x-ray” should be annotated as a medical procedure and “chest” as an anatomical structure.
1. When body location is mentioned along with the medical procedure, do not annotate the location and the medical procedure together.
  1. For example: “Left leg x-ray was normal.” In this case, only “x-ray” should be annotated as a medical procedure.
1. If a medical procedure is mentioned in a form, then we will consider such mentions as medical procedures even though they are not part of the running text.

### 2.15.x.1 Modifier: Method

#### Definition

Any mention of the method in which a procedure is conducted.

#### Examples and annotation rules

1. “X-ray was performed with IV contrast.”
  1. “IV contrast” is a method used for the “X-ray” procedure.

### 2.15.x.2 Modifier: Result

#### Definition

This modifier should be used to annotate the result of a medical procedure. The result of the procedure can have values such as “abnormal”, “normal”, “inconclusive”, and so forth.

Medical procedure results are often captured in sections of a medical record called “finding” sections. Sometimes, sections like findings occur mostly after a procedure or radiology report. The procedure result tag will only be used to label modifiers and not proper names of medical conditions. This is because most of the findings will be some medical condition detected after a radiology report or after a procedure. For example, after colonoscopy the findings are lesions, mass, bowel obstruction, diverticulitis, ulcers and annotating these findings  as a procedure result would be incorrect, as these are the main problems the patient is suffering from. We should annotate only modifiers like “positive”, “negative”, “abnormal”, “normal”, and so forth as procedure results. When a finding section has a large paragraph, it will be annotated into its relevant entity types and not its test result. 

#### Examples and annotation rules

1. “CT scan of abdomen of the patient is abnormal”
  1. “CT scan”: Procedure
  1. “abdomen”: Anatomical structure
  1. “abnormal”: Procedure result

## Category: Medication

### Definition

A drug or other preparation for the treatment or prevention of disease. Any blood products used in transfusion should also be considered in the “Medication” category. Generic medications like hypertension medications and cholesterol medications should also be annotated. Drug classes like antibiotics, antihistamines, antidepressants, and so forth should be considered Medication.

### Examples and annotation rules

1. “The patient had seen her primary care a couple of days ago, was taken off her lisinopril and digoxin.”
  1. “lisinopril “ and “digoxin” are medications.
1. Other examples of medications are “Omeprazole”, “Hydralazine'', “Prilosec”, and so forth.
1. Consider “FFP”, “Packed RBC” and other transfusion blood products as medication.  “O2” given for treatment should also be considered as medication. Blood products like “fresh frozen plasma” and “packed red blood cells” should be considered under medication.
1. The route of the medications should not be annotated together.
  1. For example: In case of “IV aspirin”, only aspirin will be annotated as Medication. 
1. Generic medications like hypertension medications and cholesterol medications should also be annotated under Medication. 
1. Drug classes like antibiotics, antihistamines, antidepressants, and so forth should also be considered under Medication.
1. Combination drugs should be annotated as a single entity. 
  1. For example: in “Fluticasone/vilanterol” is a combination drug and its trade name is Breo Ellipta. Annotate “Fluticasone/vilanterol” as a single entity.
1. Medication listed under allergy sections. 
  1. For example, “Allergies : Penicillins / Sulfonamides”. In this case, medication is neither ordered for the patient nor is a part of the patient’s continued medication list. It is just mentioned that the patient has an allergy to these medications. We should annotate:
    1. medication: "penicillins", subject: "patient", status: "other - allergy"
    1. medication: "Sulfonamides", subject: "patient", status: "other - allergy"
1. In the example: “He will continue to take aspirin, and he is advised to discontinue his antibiotics”, we should annotate:
  1. medication: “aspirin”, status: “current”, certainty assessment: “likely”
  1. medication: “antibiotics”, temporal assessment: “upcoming”, certainty assessment: “somewhat unlikely”

### 2.15.x.1 Modifier: Strength

#### Definition

The strength of the drug indicates the amount of active ingredient in each dosage.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day".
  1. “50” is the strength of the medication “Aspirin”.

### 2.15.x.2 Modifier: Unit

#### Definition

Strengths are usually quantified in units of measurements.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day".
  1. “mg” is the unit of measurement for the medication strength.

### 2.15.x.3 Modifier: Dose

#### Definition

A dose of a medicine or a drug is a measured amount of it that is intended to be taken at one time. It is the quantity of the medicine prescribed to be taken at one time.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day"
  1. “2” is the dose of the medication.

### 2.15.x.4 Modifier: Form

#### Definition

Form of medication indicates physical characteristics of the specific drug.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day".
  1. “tablets” is the form of the medication.

### 2.15.x.5 Modifier: Frequency

#### Definition

The frequency of a drug refers to how often it is taken.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day".
  1. “every day” is the frequency of the medication administration.

### 2.15.x.6 Modifier: Route

#### Definition

A route of administration is the path by which the drug is taken into the body. Route is generally classified by the location at which the drug is administered.

#### Examples and annotation rules

1. "The patient takes Aspirin 50 mg 2 tablets orally every day".
  1. “orally” is the route of the drug.

### 2.15.x.7 Modifier: Duration

#### Definition

The time until which the patient has to take the medication is known as duration of the drug.

#### Examples and annotation rules

1. "The patient should take Aspirin 50 mg 2 tablets orally every day for 2 months". 
  1. “for 2 months” is the duration during which the patient has to take the medication.

### 2.15.x.8 Modifier:   Status

#### Definition

The medication has different status like “increased”, “decreased”, “start”, “stop”, “held”, “discontinue” and so forth, and such information should be captured under the label Status.

#### Examples and annotation rules

1. “Aspirin was stopped.”
  1. “stopped” represents the status of the medicine.

### Additional rules for annotating medicine attributes

* 2.16 The relationship boundary is one sentence. Do not link entities that are related beyond one sentence.

* 2.17 If there are multiple mentions of a drug in a sentence, then all the mentions should be related to its attributes.

  * For example, “stool softener (colace) 2mg”. In this case, “2” as strength and “mg” as unit should be related to both “stool softener” and “colace”.

* 2.18 Attributes are not standalone and should be related to medication.

* 2.19 If the status of a medication is “changed”, “switched”, “stop”, “increase”, “decrease”, it should be annotated under the Status attribute.

* 2.20 The final dose of a medication should be annotated and not the previous dose.

* 2.21 When puffs, drops, or sprays are mentioned with their dosage, then dosages like “1 drop”, “2 puffs”, and so forth  are annotated with 1 as the dose and drop as the form.

* 2.22 For duration, annotate the preposition for in “for 2 weeks” together instead of only annotating “2 weeks”.

* 2.23 If the time is written as “at bedtime”, then annotate the preposition as well.

## Category: Substance abuse

### Definition

This entity type describes the abuse caused by a psychoactive substance that affects brain function and results in alterations in mood, consciousness, perception, cognition, or behavior. This category covers alcohol, cocaine and other substances of abuse. It also covers smoking and IV drug use.

### Examples and annotation rules

1. “The patient has a history of alcohol and cocaine abuse.”
  1. “alcohol and cocaine abuse” will be annotated as substance abuse.
1. Substance abuse category includes abuse, dependency, or use of psychotropic substances like alcohol, IV drugs, and nicotine.
1. If multiple substances are mentioned with abuse or dependencies, then the whole phrase should be annotated.
  1. For example: “The patient is cocaine and alcohol dependent.” In this case, “cocaine and alcohol dependent” should be annotated as substance abuse.

## Category: Patient status

### Definition

The annotation of status is dependent on the keyword present in the document. There are times when the document is missing such information. In such cases, the annotator must not annotate the status. Do not confuse this scenario with an unclear entity type. “Unclear” should be only used when it is documented.

### 2.23.x.1 Living status

This category describes the living condition of the patient. There are multiple possibilities and hence this category has 4 subtypes that cover all possibilities.
* Living status:
* Alone
* Family
* Assisted living
* Other

In case of living status, the entity that represents the living status will be annotated.
* For example: “Patient lives with family.” In this case, “family” will be assigned the tag “Family”.
* For example: “Patient lives with friends.” In this case, “friends” will be assigned the tag “Other”.

### 2.23.x.2 Follow-up status

This category describes the follow-up status of the patient. “Unclear” should be annotated when it is documented and is not to be confused with no related documentation.

Followup status:
* Follows up
* Does not follow up
* Unclear - Follow up

### 2.23.x.3 Compliance status

This category describes the compliance status of the patient. There are times when the patient is not compliant due to a specific reason, and such reasons should be also annotated as the reason for non-compliance.

Compliance:
* Compliant
* Non-compliant
  * Reason for non-compliance

Words or phrases like “not taking his/her medication” depicts non-compliance, and so “not taking his/her medications” will be annotated as non -compliant.

### 2.23.x.4 Monitoring status

This category describes monitoring status of the patient.

Monitoring status:
* Regular
* Irregular
* Unclear- Monitoring

For “Follow-up status” and “Monitoring status”, there are times when nothing is mentioned in the related document. In that case, nothing will be annotated as there will be no keyword to assign a tag.

* When keywords appear in the document, those keywords will be assigned the entity type. For example: “The patient follows up with PCP and lives with family.” In this example, “follows up” will be annotated with the entity type “Follows up” and the word “family” will be annotated as “Family”.
* Example 2: “It is unclear whether he follows up with a physician on a routine basis.”
  * In examples like this one, the tag “unclear” will be annotated.
