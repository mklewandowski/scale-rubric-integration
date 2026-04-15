#  Integration Notes
The integration goal is to access C2C teacher, student, and performance data and provide a simplfied version of the SCALE Science Learning-Centered Rubric site which mainly provides summary and assessment information.

## Site Features to Preserve
to do

## New Site Features
to do

## What is Needed to Display Individual and Group Assessment Pages

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
