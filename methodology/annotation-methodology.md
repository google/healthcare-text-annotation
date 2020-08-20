# 1. Annotation methodology

## 1.1 Phase 1 - Overview and training

* Required participants: annotators, domain expert, and computational linguist.
* In this phase, the annotators receive an overview of the project.
* Annotators go through the guidelines and discuss any disagreements internally so that everyone is aligned.
* The domain expert gives a demonstration of the annotation tool along with some sample annotations.
* The domain expert and computational linguist resolve disagreements regarding the understanding of the guidelines.

## 1.2 Phase 2 - Sample annotations

* After the annotators have been trained according to Phase 1, each pair of annotators will be given 10 documents (this number may vary to accommodate a representative data sample). Each pair of annotators annotate together to ensure the same understanding toward the task.. By taking this approach of working in pairs, annotators are aligned and resolve their disagreements internally.
* After annotating the first 5 documents, the domain expert will review the annotations, solve any disagreements, and confirm the annotators’ understanding.
  * If the annotation quality matches the annotation agreement threshold (for example, greater than 95%, although the threshold might vary based on the project), the annotators proceed to annotate the remaining 5 documents and have the documents reviewed by the domain expert.
  * If the quality result is not as expected, the annotators will undergo training again and repeat the steps outlined in Phase 2.

## 1.3 Phase 3 - Annotation task

* Annotation is a mentally taxing task, so annotators occasionally miss annotating some of the entities, especially when a document contains many entities and relations. Inter-Annotator Agreement (IAA) should be computed to ensure quality of data annotations.
* Each annotator receives a different pair of documents, with a 5% overlap between them, (may go as high as 20%) to calculate the IAA.
* The domain expert and computational linguist will be involved with the team and will resolve disagreements and questions. They will also review the agreements in the annotations and suggest changes, if necessary.
* Disagreements stemming from the annotation guidelines should be documented and resolved as they occur.
* If there is any disagreement between the annotators, the domain expert and the linguist will make a decision and document the issues and conclusion so that other groups can refer to the decision if there is a similar question. The groups can then resolve the disagreement internally, thus saving time and improving accuracy.
* Documents that have an IAA score of less than 95% will be reviewed and the necessary changes in annotations will be made to achieve a greater than 95% IAA score.

## 1.4 Annotation format

For the purpose of these annotation guidelines, annotation examples are represented in a JSON payload. Each medical entity is assigned an ID and additional fields like certainty assessment, temporal assessment, and subject values. Below is the JSON representation of annotations for a document containing only the text, “The patient has a personal history of asthma.”

```json
{
      "entities" : [ {
          "entity_id" : 1,
          "text_extraction" : {
              "text_segment" : {
                  "start_offset" : 38,
                  "end_offset" : 42
              }
          },
          "entity_type" : "SEVERITY",
          "temporal_assessment" : "PERSONAL_HISTORY",
          "certainty_assessment" : "LIKELY",
          "subject" : "PATIENT"
      }, {
          "entity_id" : 2,
          "text_extraction" : {
              "text_segment" : {
                  "start_offset" : 45,
                  "end_offset" : 51
              }
          },
          "entity_type" : "PROBLEM",
          "temporal_assessment" : "PERSONAL_HISTORY",
          "certainty_assessment" : "LIKELY",
          "subject" : "PATIENT"
      }],
      "relationships" : [ {
          "relationship_type" : "PROBLEM_SEVERITY",
          "subject_id": 1,
          "object_id": 2
      }],
      "text_snippet": "The patient has a personal history of severe asthma."
    }
}
```

