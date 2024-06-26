
## Project Information

|   Info      |        Description     |
| ----------- | ---------------------- |
|  Title      |      UIUC Coursehub                                                                                      |
| System URL  |                         35.209.141.178                                                                   |
| Video Link  |     https://drive.google.com/file/d/17bK7243IMjBegK_UcPAno-DU3LD1dKd5/view?usp=sharing                   |

## Project Summary
Our project will be a web application which displays GPAs, general statistics for a variety of courses offered at UIUC across different departments from Summer 2010 to Fall 2021, along with student reviews and ratings on those courses on various metrics. This data will be accessed through a database in the backend.

Users will be able to select a course, its year, its term, its department, an instructor, or many different combinations of these categories to display GPAs and reviews for a course. This would require a database schema which fits the normal forms, and processes for rejecting search requests which cannot be possible. Additionally, to display average GPAs and their relevant statistics, math must be done, as well as potentially complex graphing.

Furthermore, users can select a course section and write reviews for the course, being displayed if another user who searches for the course clicks on it. Logic should be implemented where a particular user cannot submit multiple reviews for one course section. Instead, after every subsequent review from the first, the previous gets overwritten in favor of the new review.
