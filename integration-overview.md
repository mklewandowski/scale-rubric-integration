#  Integration Notes
The integration goal is to access C2C teacher, student, and performance data and provide a simplfied version of the SCALE Science Learning-Centered Rubric site which mainly provides summary and assessment information.

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

## What is Needed to Display Individual and Group Assessment Pages
At present the SCALE Rubric site displays assessment information based on the scoring information stored in the user state. This is done for a single rubric and class (a class is defined as a roster of 1 or more sutdents). No inter-class or inter-rubric performance data is aggregated or provided to the user. The current assessement information includes:
- per-student individual performance level
- individual performance level percentages for all questions across class
- individual performance level percentages for each question across class
- individual performance level percentages for each category of each question across class
- per-student group performance level
- group performance level percentages for all questions across class
- group performance level percentages for each question across class
- group performance level percentages for each category of each question across class
- specific details about each question (used in exported reports)
- per-student notes for each question (for this integration, we will likely want to exclude notes)

For the SCALE Rubric site to display assessment data, the following is needed:
- classroom information including:
  - class name
  - class roster containing student names and student ids
- current rubric (some way to identify which rubric we are using, an id, a chosen static name, etc.)
- scoring information. For each student, the total points for each category in each question in the rubric. Question and question categories must also have an id or some way to map them to the question information stored for each rubric.


## How User State is Currently Stored
The SCALE Rubric site stores basic session information (class name, student list, user input) using the browser’s web storage API (local storage). This information is loaded when the site is initially loaded and saved as the user makes changes. Content is ONLY saved on the local system in a browser folder. There is a 5MB limit to what we can store in local storage. We compress and decompress data saved in local storage to limit the chance that we hit this 5 MB limit. The compressed data is not in human readable form so any student information stored with the browser is not readable.

https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API

We store the following entries in local storage:
- `classInfo`: the class information containing a list of classes. Each class has a name, an id, and a list of students.
- `userInput`: the user input for all classes and all rubrics (which checkboxes were checked when scoring a student).
- `accessibleText`: if we show color text or black and white text.
- `appOptions`: contains user options (e.g., state of show all students/dimensions), eventually we want to merge accessible mode into this.
- `privacyAccepted`: if the user accepted the privacy policy.

We have 5MB total of local storage that can be used. For this reason, data in local storage is both minified and encrypted before it is saved.
