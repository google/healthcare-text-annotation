# Rules for annotating Temporal Assessment, Certainty Assessment, Subject

1. We will consider the following as default values and not annotate them. This is because in most cases such values are not mentioned with any specific keywords that we can relate it with, and it is generally considered that the subject is “patient”,the entity is “likely”, and the status is “current”. 
  * Certainty Assessment: Likely (Default)
  * Subject: Patient (Default)
  * Temporal Assessment: (Current)
    * Example 1: “Gastrointestinal: abdominal pain, nausea, vomiting”. This sentence has no keywords to annotate the “Likely, Patient or Current status”, but the sentence does mean that as it is a part of a physical examination result.
    * Example 2: “He has hypertension”. In this case, there are no keywords to annotate the subject, certainty assessment, and temporal assessment of the entity “Hypertension”.
2. The temporal assessment, certainty assessment, and subject will only be annotated if it is related to any medical entity present in the sentence.
3. Sections like “Family history”, “Past medical history”, “Social history”, and “Past surgical history” should be used as trigger words to detect the status of an entity.
4. “Subject, Certainty Assessment, Temporal Assessment” should be assigned to the entity (Problem, Procedure, Medicine).
  * For example: “Father has hypertension.” In this case, “hypertension” should be labeled as a medical condition along with “temporal assessment: Family history” and “subject: Family member”.
5. If any medication is listed under the allergy section, or if it is documented that the patient's family member has an allergic reaction to some medicine, then that medicine should be assigned a temporal assessment “other- allergy”.
  * For example: “The patient has an allergy to penicillin”. In this case, penicillin should be annotated as medicine and should be given “Temporal Assessment: other allergy, Subject: patient, Certainty Assessment: Likely.”
6. In the case of a conditional situation, such as “The doctor said that the patient should also come back if he develops pain, fever, nausea”, “if he develops” is the trigger for conditional and hence “pain”, “fever”, and “nausea” should be annotated as conditional.
7. There are instances where a disease is documented as related to a family member. For example: “Father has hypertension”. In this example, we should annotate “hypertension” as disease, with “Temporal assessment: family history”, and “subject: family member”. 
