# 2.27 Defining entity relations

## Anatomical body structure <> Medical device

This relationship provides information about the medical devices related to anatomical structure.
For example: “The patient has an RCA stent in place.”
Here, “RCA” is an anatomical structure and “stent” is a medical device. Adding a relationship between anatomy and a medical device helps us to retain the context of the sentence.

## Medical condition <> Medical device

This relationship is specifically for a medical condition and the device used to address that condition.
For example: “The patient has a fracture and hence a splint was placed.” “Splint” should be annotated as a medical device and should be assigned a relationship with the medical condition.

**Note**: If any device occurs alongside multiple entity types, then we should assign multiple relationships.

For example: “The patient has a right hand splint for his fracture.”
Here, “splint” - medical device - should be related with “fracture” and anatomical structure.

## Medical condition <> Medication

Generally, medications present in the document have some reference or relationship with the medical condition. There are exceptions, as there are times when a list of medications are documented in reference to the chronic conditions of the patient which may or may not be documented in the record.

For example: “The patient is on insulin for his diabetes.”
In this case, “insulin” is a medication used to treat diabetes and, so both these entity types should be related.

## Medical condition <> Procedure

When a procedure is performed to treat a specific condition, then this relationship should be used. Imaging performed to detect a specific problem should also be considered.

For example: “The patient undergoes CPAP therapy for his sleep apnea.”

Here, “CPAP therapy” is used for “his sleep apnea”. By assigning a relationship between procedure and problem, we can extract the context of the sentence.

## Anatomical body structure <> Medical condition

Assign this relationship when a medical condition is mentioned as being related to a specific anatomical body region.When a medical condition is mentioned as being related to a specific anatomical body region, then this relationship should be assigned.

For example: “The patient has pain in her hand.”
Here, “pain” is present in the patient’s hand and hence “pain” and “hand” should be labeled as related.

## Anatomical body structure <> Medical procedure

This type of relationship should be assigned when a specific procedure is carried out in an anatomical structure.

For example: “The resection of the gallbladder took place.”
“resection” (medical procedure) of the anatomy part “gallbladder” (anatomical body structure) is documented in the medical record. By labeling the relationship between Medical procedure and Anatomical body structure, we can extract a deeper level of information regarding the patient’s treatment plan.
