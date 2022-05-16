## Application Development Team

The Application Development Team (App Dev Team) maintains a suite of services supporting the integration of several applications including AIM, CARL, CAP, ImageRight, ConceptOne and Titan Imaging. Our goal is ensure each of these systsm has access to the data it requires to support its business objectives.

### Our Services

Our initial purpose was to enable data to flow in and out of AIM for external systems. We built the "AIM API" that consists of a set of REST-based APIs that allow external systems to send data to AIM via HTTP requests. We also support the AIM Change Watcher which tracks changes in the AIM database and broadcasts messages of changes made to entities. These services allow external systems to keep in-sync with changes being made in AIM and to provide updated/new information to AIM, such as policy data and invoice data.

We also support the TaxSmart API which provides an HTTP endpoint for calculating taxes. It replicates the tax calculations in AIM allowing external systems to provide proper tax data in the policy and invoice data they send to the AIM API.

Over time we have built a series of applications to enhance or replace features in AIM to support the business needs of our clients:

- Universal Clearance: A replacement of the AIM process for clearing new business. It ingests submission and policy data from all quote/rate/bind applications in order to clear against all these systems, rather than only data available in AIM.

- Market Smart: A replacement for the AIM process of marketing policies to carries for quotes on new business and renewals.

- Renewal Solicitation: A means of streamlining processing renewals and soliciting business with agents.

- Pro Exec: A means of capturing additional data on pro exec policies prior to binding in order to track and analyze the pro exec market.

- Syndicate Tracking: A means of capturing syndicate data on policies to record the percentage of the overall risk for a syndicate on a policy.

- Tax Exchange: A means of exchanging tax information with state surplus lines associations for invoices processed in AIM.

Over the last year, we took on new responsibilities for converting documents in AIM, Smith and instances of ImageRight into Titan Imaging. This was in support of Project Fusion that moves the processing of policies into Titan-CL from AIM, Smith, ASL and MFS. As part of that work, we also support the RT Way initiative by providing a data warehouse of policy and document data from AIM and ImageRight.

### Our Process

We follow the Scrum methodology for managing our work. We follow a two week cadence for completing each Sprint. Every second Wednesday we have a Sprint Planning meeting to determine what our team will commit to in the Sprint. 

We enter stories in JIRA and use it to help us refine the stories, until they are ready to be worked by the team. The team conducts Product Backlog Refinement (PBR) meetings twice each week to meet with the product owner and discuss the details of stories in the backlog. The team also conducts 'Technical' BPR meetings where the developers meet to discuss technical aspects of stories and determine if we have sufficient understanding of how to complete the work.

When a story is brought into a Sprint, the team breaks each story down into a series of tasks, each task alloted an estimated number of hours to complete. We use JIRA to track the hours remaining on a task as our burn down chart. It provides us with a simple indicator of how well we are progressing within a Sprint.

When the work for a story is sufficiently complete, the team arranges to demonstrate the work with the product owner. These 'Demo to Production Owner (PO)' tasks help ensure the work is meeting expectations as early as possible in the Sprint, to avoid receiving feedback too late in the Sprint.

As each story is developed, we conduct a series of tests starting with the developers writing unit tests. We expect each change to the system to be accompanied by supporting unit tests (this typically does not apply to UI code changes). Once the work is deployed to the shared development or test environments, the team writes one or more integration (end-to-end) tests to ensure each acceptance criteria in the story is satisfied. The team also conducts a series of business tests that verify the system behaves as expected, from a user's perspective. The results of these business tests are recorded in TestRail.

Once all the tests are complete, the product owner signs off on the work and it is approved to be released to Production. 

At the end of each Sprint we have a retrospective meeting where we discuss how our process faired during the sprint, and if we feel any changes can be made to improve it.

### Technical Practices

Trunk-Based Development

Our team develops software using a trunk-based approach. There is a single master branch of code and all changes are are made to this branch. We do not maintain any other development branches. We strive to only create small, short-lived feature branches. A developer (or programming pair) creates a branch, codes and tests their changes, and uses a Pull Request style code review to merge the changes into the master branch. Our goal is for these feature branches to live no more than a single day, in other words, each developer should be committing changes to the master branch at least once per day. 

Keeping the branches short-lived with a small set of changes allows us to get feedback quickly and to not overwhelm team members with large sets of complicated changes.

Feature Flags

Unit Testing

Code Reviews

Continuous Integration (CI)

Continuous Delivery (CD)
