#  Integration Notes
The integration goal is to access C2C teacher, student, and user input data (the scores) and provide a simplified version of the SCALE Science Learning-Centered Rubric site which mainly provides performance summary information.

## Site Features to Preserve
- load user state and populate website from state
- subset of rubric page - only the group performance and individual performance sections 
- subset of how-to page - only content related to group performance and individual performance sections
- about page

## New Site Features
- integrated mode that hides non-preserved site features
- load user state from C2C API calls
- add timestamp display that shows when the current state was generated
- add warning "check back in a few minutes" if no user state is available
- report to user if performance information is not available for a specific rubric

## What is Needed to Display Individual and Group Performance Pages
At present the SCALE Rubric site displays performance results based on the scoring information stored in the user state. This is done for a single rubric and class (a class is defined as a roster of 1 or more students) at a time. The user selects a class and a rubric, and performance information is displayed. No inter-class or inter-rubric performance data is aggregated or provided to the user. The current performance information includes:
- per-student individual performance level
- individual performance level percentages for all questions across class
- individual performance level percentages for each question across class
- individual performance level percentages for each dimension/category of each question across class
- per-student group performance level
- group performance level percentages for all questions across class
- group performance level percentages for each question across class
- group performance level percentages for each dimension/category of each question across class
- specific details about each question (used in exported reports)
- per-student notes for each question (for this integration, we will likely want to exclude notes)

For the SCALE Rubric site to display performance results, the following is needed from a data source:
- classroom information including:
  - class name
  - class roster containing student names and student ids
- current rubric (some way to identify which rubric we are using, an id, a chosen static name, etc.)
- scoring information. For each student, the total points for each dimension/category in each question in the rubric. ***Question and question dimensions/categories must also have an id or some way to map them to the question information stored for each rubric.***


## How User State is Currently Stored and Retrieved
The SCALE Rubric site stores basic session information (class name, student list, user input) using the browser’s web storage API (local storage). This information is loaded when the site first loads and saved as the user makes changes. Content is ONLY saved on the local system in a browser folder. There is a 5MB limit to what we can store in local storage. We compress and decompress data saved in local storage to limit the chance that we hit this 5 MB limit. The compressed data is not in human readable form so any student information stored with the browser is not readable.

https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API

We store the following entries in local storage:
- `classInfo`: the class information containing a list of classes. Each class has a name, an id, and a list of students.
- `userInput`: the user input for all classes and all rubrics (which checkboxes were checked when scoring a student).
- `accessibleText`: if we show color text or black and white text.
- `appOptions`: contains user options (e.g., state of show all students/dimensions), eventually we want to merge accessible mode into this.
- `privacyAccepted`: if the user accepted the privacy policy.

Each class, student, rubric, question, question dimension, and question dimension point (the individual checkbox that is checked/unchecked and represents a single point in the scoring) has a unique id. Internally, the userInput is transformed into a hierarchical JSON object with ids as keys using the following TypeScript definitions:

```
export interface UserInput {
  cls: UserInputClasses;
}
export interface UserInputClasses {
  [key: string]: UserInputClass;
}
export interface UserInputClass {
  [key: string]: UserInputStudent;
}
export interface UserInputStudent {
  [key: string]: UserInputRubric;
}
export interface UserInputRubric {
  [key: string]: UserInputQuestion;
}
export interface UserInputQuestion {
  [key: string]: UserInputDimension;
}
export interface UserInputDimension {
  [key: string]: UserInputPoint | string;
}
export interface UserInputPoint {
  v: 0 | 1;
}
```

An example of this local storage state [can be viewed here](local-storage-user-state-data.json).

## Integration Proposal for Structuring User State  
As mentioned in "What is Needed to Display Individual and Group Performance Pages", the minimum information required to display Individual and Group Performance Pages is classroom information (class name, student roster), the current rubric, and per-student scoring information for all questions of the current rubric. 

### Option 1: ID-based Mapping Model
We could use an ID-based mapping model for the rubric, question, dimension, and points and link this to the internal rubric information stored in the SCALE Rubric site. This will massively limit how much information that we pass from the C2C platform to the SCALE Rubric site, but it does place a burden on the authoring team to ensure that ids for rubric information in the C2C platform match ids for rubric information in the SCALE Rubric site. 

***RISK: Coordinating a unified id system used in both the C2C platform and SCALE Rubric site could be challenging.***

Here is an example of the data passed from a C2C API endpoint:
```
{
  "rubricId": "id1",
  "class": {
    "name": "test class",
    "roster": [
      {
        "name": "john doe",
        "id": "s1"
      },
      {
        "name": "jane doe",
        "id": "s2"
      }
    ]
  },
  "userInput": [
    {
      "studentId": "s1",
      "questions": [
        {
          "id": "q1",
          "dimensions": [
            {
              "id": "d1",
              "points": [
                {
                  "id": "p1",
                  "value": 1
                },
                {
                  "id": "p2",
                  "value": 0
                }
              ]
            },
            {
              "id": "d2",
              "points": [
                {
                  "id": "p1",
                  "value": 1
                },
                {
                  "id": "p2",
                  "value": 0
                }
              ]
            }
          ]
        },
        {
          "id": "q2",
          "dimensions": [
            {
              "id": "d1",
              "points": [
                {
                  "id": "p1",
                  "value": 1
                },
                {
                  "id": "p2",
                  "value": 0
                }
              ]
            },
            {
              "id": "d2",
              "points": [
                {
                  "id": "p1",
                  "value": 1
                },
                {
                  "id": "p2",
                  "value": 0
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "studentId": "s2",
      "questions": [
        {
          "id": "q1",
          "dimensions": [
            {
              "id": "d1",
              "points": [
                {
                  "id": "p1",
                  "value": 0
                },
                {
                  "id": "p2",
                  "value": 1
                }
              ]
            },
            {
              "id": "d2",
              "points": [
                {
                  "id": "p1",
                  "value": 0
                },
                {
                  "id": "p2",
                  "value": 1
                }
              ]
            }
          ]
        },
        {
          "id": "q2",
          "dimensions": [
            {
              "id": "d1",
              "points": [
                {
                  "id": "p1",
                  "value": 0
                },
                {
                  "id": "p2",
                  "value": 1
                }
              ]
            },
            {
              "id": "d2",
              "points": [
                {
                  "id": "p1",
                  "value": 0
                },
                {
                  "id": "p2",
                  "value": 1
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### Option 2: Self-Describing Model
Instead of using matching ids that are stored in both the C2C platform and the SCALE Rubric site, we would use the above JSON format for the user input (the scoring), and also include ALL necessary information to construct and display the Individual and Group Performance Pages. This means the C2C API will also need to pass data about the Rubric and each question. Essentially the result from the C2C API would be a fully self-describing model of the data.   

***RISK: Passing the full rubric definition in every API call is redundant and increases the payload size significantly. This also places extra burden on the engineering team to construct and send additional information.***

For the rubric, instead of passing a simple id:
```
"rubricId": "id1",
```
We would need to pass an object with additional information:
```
"rubric": {
  "name": "Ancient Artifacts",
}
```

The same would be true for questions. We would need to include additional information about each question which could be used to build the performance pages:
```
  "questions": [
    {
      "q_id": "1",
      "short_name": "GQ:3",
      "name": "Group Question 3",
      "is_group": true,
      "question_content": "First, use the information about the organisms (from &lt;b&gt;Page 3&lt;/b&gt;) to develop food web models on &lt;b&gt;Page 2&lt;/b&gt;:\na. Add &lt;u&gt;arrows&lt;/u&gt; to each drawing to show how &lt;b&gt;matter &lt;/b&gt;is&lt;b&gt; &lt;/b&gt;transferred between organisms.\nb. Add &lt;u&gt;captions&lt;/u&gt; to each arrow to explain &lt;b&gt;feeding relationships&lt;/b&gt; between organisms.",
      "performance_outcomes": "&lt;b&gt;&lt;span style='color:#2563c2'&gt;Develop a model that describes&lt;/b&gt;&lt;/span&gt;&lt;b&gt;&lt;span style='color:#0b8871'&gt; how matter moves through a &lt;/b&gt;&lt;/span&gt;&lt;b&gt;&lt;span style='color:#d56717'&gt;healthy “food web” of organisms in an ecosystem. &lt;/b&gt;&lt;/span&gt;",
      "student_example": "&lt;b&gt;[For clarity, this example shows just the parts of the Page 2 model that are relevant to Question 3, but students' models will also have more components related to Question 4.] &lt;/b&gt;&lt;i&gt; &lt;/i&gt;",
      "dimensions": [
        {
          "d_id": "1",
          "type": "science_engineering_practices",
          "dimension_alignment": "&lt;b&gt;Developing and Using Models\n&lt;/b&gt;Develop a model to describe phenomena.",
          "dimension_alignment_header": "Developing and Using Models",
          "description_performance": {
            "points": [
              {
                "p_id": "1",
                "prompt": "Develops a model, including:",
                "content": "Adds arrows showing interactions between organisms."
              }
            ]
          },
          "total_points": 1
        },
        {
          "d_id": "2",
          "type": "crosscutting_concepts",
          "dimension_alignment": "&lt;b&gt;Systems and System Models\n&lt;/b&gt;A system can be described in terms of its components and their interactions.\n&lt;b&gt;Energy and Matter&lt;/b&gt;\nMatter is transported into, out of, and within systems. \n\n",
          "dimension_alignment_header": "Systems and System Models",
          "description_performance": {
            "points": [
              {
                "p_id": "1",
                "prompt": "",
                "content": "All relevant feeding relationships in the ecosystem are represented."
              },
              {
                "p_id": "2",
                "prompt": "",
                "content": "Direction of arrows accurately represents direction of matter transfer (e.g., arrow from rivergrass towards snail, arrow from snail towards limpkin)."
              }
            ]
          },
          "total_points": 2
        },
        {
          "d_id": "3",
          "type": "disciplinary_core_ideas",
          "dimension_alignment": "&lt;b&gt;LS2.A:  Interdependent Relationships in Ecosystems\n&lt;/b&gt;The food of almost any kind of animal can be traced back to plants. Organisms are related in food webs in which some animals eat plants for food and other animals eat the animals that eat plants. &lt;span style='color:#A0A0A0'&gt;Some organisms, such as fungi and bacteria, break down dead organisms (both plants or plants parts and animals) and therefore operate as “decomposers.” Decomposition eventually restores (recycles) some materials back to the soil. &lt;/span&gt;Organisms can survive only in environments in which their particular needs are met. &lt;span style='color:#A0A0A0'&gt;A healthy ecosystem is one in which multiple species of different types are each able to meet their needs in a relatively stable web of life. Newly introduced species can damage the balance of an ecosystem. &lt;/span&gt;\n&lt;b&gt;LS2.B:  Cycles of Matter and Energy Transfer in Ecosystems\n&lt;/b&gt;Matter cycles &lt;span style='color:#A0A0A0'&gt;between the air and soil and &lt;/span&gt;among plants, animals, &lt;span style='color:#A0A0A0'&gt;and microbes as these organisms live and die. Organisms obtain gasses, and water, from the environment, and release waste matter (gas, liquid, or solid) back into the environment. &lt;/span&gt;\n",
          "dimension_alignment_header": "LS2.A:  Interdependent Relationships in Ecosystems",
          "description_performance": {
            "points": [
              {
                "p_id": "1",
                "prompt": "Adds captions describing how interactions transfer matter in the ecosystem, including:",
                "content": "Describes how some animals eat plants (e.g., &lt;i&gt;snails eat grass&lt;/i&gt;)."
              },
              {
                "p_id": "2",
                "prompt": "",
                "content": "Describes how some animals eat animals (e.g., &lt;i&gt;limpkins eat snails&lt;/i&gt;)."
              }
            ]
          },
          "total_points": 2
        }
      ]
    }
]
```