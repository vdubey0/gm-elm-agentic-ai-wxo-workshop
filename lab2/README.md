# Lab 2: Requirement Change Impact Analysis

## Table of Contents
- [1. Introduction](#introduction)
- [2. Accessing watsonx Orchestrate UI](#ui)
- [3. Building the Requirement Change Solution](#requirement-change-soln)
    - [3.1. Building the Message Agent](#building-message-agent)
    - [3.2. Building the Requirement Change Agent](#building-req-change-agent)
- [4. Deploy Your Agent (optional)](#deploy)
- [5. Summary](#summary)


### Downloadables

Ensure you download these below files and keep them handy since we will be using them as part of the lab.

1. [OpenAPI Spec](downloadables/openapi.yaml) `openapi.yaml`

2. [Contacts Mapping](downloadables/contacts.csv) `contacts.csv`

<details open id="introduction">
<summary><h2>1. Introduction</h2></summary>

![alt text](images/updated_arch.png)

This lab focuses on automating **requirement change impact analysis and stakeholder notifications** using watsonx Orchestrate. The goal is to help system engineers understand the downstream effects of requirement changes in DOORS Next and proactively notify affected artifact owners — all through an AI-driven, multi-agent workflow integrated with Engineering Lifecycle Management (ELM).

The scenario centers on a **System Engineer** who has modified one or more requirements in DOORS Next. When a requirement changes, it can cascade through a complex web of downstream requirements and test cases — each owned by different team members. Manually tracing these impacts and notifying the right people is tedious and error-prone.

In this lab, you will build an AI-powered workflow within watsonx Orchestrate that:

* Accepts one or more **requirement IDs** as input from the engineer.
* Performs **impact analysis** to identify all downstream requirements and test cases affected by the change.
* Runs **conflict analysis** to detect potential mismatches between parent and child requirements.
* Generates **targeted Slack notifications** for each affected artifact owner.
* Delivers those notifications automatically via **Slack direct messages**.

The solution uses two collaborating agents — a **Requirement Change Agent** that orchestrates the analysis workflow, and a **Message Agent** that handles Slack communication — coordinated seamlessly within watsonx Orchestrate.

By automating requirement change impact analysis and notifications, engineering teams can:

* Eliminate manual tracing of downstream dependencies.
* Detect requirement conflicts early before they propagate.
* Ensure every affected owner is notified promptly and consistently.
* Accelerate change management while maintaining full traceability across the ELM lifecycle.

[← Back to Table of contents](#table-of-contents)
</details>

<details open id="ui">
<summary><h2>2. Accessing watsonx Orchestrate UI</h2></summary>

1. To access the watsonx Orchestrate console, go to the [Resources list on the IBM Cloud homepage](https://cloud.ibm.com/resources).

    ![alt text](images/img43.png)

2. Expand the **AI / Machine Learning** section and select the resource that has **watsonx Orchestrate** in the Product column, as shown above. Then click **Launch watsonx Orchestrate**.

    ![alt text](images/img44.png)

3. This opens the watsonx Orchestrate console.

    ![alt text](images/img45.png)

4. Hit the hamburger menu on the top left corner and select **Build**

    ![alt text](images/img46.png)

[← Back to Table of contents](#table-of-contents)

<details open id="requirement-change-soln">
<summary><h2>3. Building the Requirement Change Solution</h2></summary>


In this section, we will walk through the steps to build the Requirement Change Solution using multiple collaborating agents in watsonx Orchestrate.

This solution consists of two agents:

1. **Message Agent:**  
   This agent is responsible for delivering Slack direct messages to the appropriate artifact owners. It uses a contacts knowledge base to resolve employee IDs to Slack member IDs, and sends targeted notifications on behalf of the orchestrating agent.

2. **Requirement Change Agent:**  
   This agent orchestrates the full impact analysis workflow. It accepts requirement IDs as input, invokes the Impact Analysis and Conflict Analysis tools to identify downstream effects and conflicts, and delegates Slack notifications to the Message Agent.

Let us begin by building the Message Agent first.

<details open id="building-message-agent">
<summary><h3>3.1. Building the Message Agent</h3></summary>

1. Go to the **Build Agents** tab and click **Create Agent**.
![alt text](images/img1.png)
2. Since we will be creating an agent for this, select the `Create from scratch` option.
![alt text](images/img2.png)
3. We will be building this agent from scratch, therefore enter the provided name and description to the agent.
    - **Name:** This describes the name of the Agent. Agents must have a unique name. Please prefix the agent name with **your initials followed by two digits** (e.g., `AB01`, `NF23`, etc.).

        Copy and paste the below format in the Name field, replacing the placeholder with your own initials and digits:
        ```
        <YOUR_INITIALS><TWO_DIGITS> Message Agent
        ```

    - **Description:** This describes what tasks the agent will be performing, this will be useful when we are building a multi agentic solution when agents need to discover suitable agents based on the task in hand. Copy and paste the below in the Description field.
        ```
        This agent specializes in taking in messages and sending them in Slack via DM to the appropriate owners.
        ```
        Once you are done entering the above details, click **Create**.
![alt text](images/img3.png)

4. On the next screen, you can configure additional details about your agent. You can select the Large Language Model the agent uses and the style. For this agent, select `GPT-OSS 120B` and the agent style as `React`.
![alt text](images/img4.png)
5. Scroll down to the **Knowledge** section add hit **Add source**. 
![alt text](images/img5.png)
6. In the **Add source** dialog box, hit the **New Knowledge** button.
![alt text](images/img6.png)
7. You will now see a **New Knowledge Source** dialog. Scroll all the way down and select **Upload Files**. Once **Upload Files** is selected, click **Next**.
![alt text](images/img8.png)
8. You will now see the **Add Knowledge** dialog where you will be prompted to upload a knowledge file. You are to download `contacts.csv` from [here](downloadables/contacts.csv) and upload it in this screen. Once you have uploaded the file, hit **Next**. 
![alt text](images/img9.png)
9. You will now see a section for **Knowledge details** where you are to fill out the details for the knowledge base:
- **Name**: `Contacts`
- **Description**: 
```
This knowledge base contains map from employee id to their Slack user id. This document is to be used to when the agent needs to retrieve the Slack user id in order to send a notification to them`
```
![alt text](images/img10.png)
10. Once the fields have been filled out, click **Save**. You should be taken back to the main agent page.
![alt text](images/img11.png)
11. Scroll one section down to the **Toolset** section and find the **Tools** subsection. Click the **Add Tool** button. 
![alt text](images/img12.png)
12. You will now see the **Add Tool** dialog. In the bottom row, click **OpenAPI**.
![alt text](images/img13.png)
13. You will now see the **Upload Files** dialog. Click on **Drag and drop an OpenAPI file here or click to upload**. You are to upload `openapi.yaml` from [here](downloadables/openapi.yaml).
14. Click **Next**.
![alt text](images/img14.png)
15. In the next screen, you will see a list of tools that we have pre-built for you. Select **Send Slack Direct Message** and and hit **Done**.
![alt text](images/img15.png)
16. You will now be taken back to the main Agent page. You should see the tools imported under the **Tools** section. However, let's follow best practices to add more appropriate names/descriptions to these tools.
17. On the right of **Send Slack Direct Message**, click the three dots and then click **Edit Details**.
![alt text](images/img16.png)
18. In the ensuing pop-up, update the name to **Send Slack Direct Message** and add the description: 

```
This tool sends Slack direct messages to impacted artifact owners using Slack member IDs
```

Then hit **Done**.
![alt text](images/img17.png)
19. Your screen should currently look like this:
![alt text](images/img18.png)
20. Now, we will specify the **Behavior** of the agent. This section specifies the order of operation that the agent will take. We have configured the following **Behavior** for the purposes of this agent. Scroll down to the **Behavior** section and copy-paste the following text.

```
You will take input in the following format:
{
<recipient_contributor_username> : <message>,
<recipient_contributor_username>: <message>,
<recipient_contributor_username> : <message>
}

For each entry in this input dictionary, you are to do the following:
1. The key (i.e. <recipient_contributor_username>) is the employee_id. Store the employee_id.
2. Use the employee_id to get the member_id using the Contacts knowledge base.
3. Use the tool Send Slack Direct Message with member_id = <member_id obtained in step 2> and text= <message from input>.
```
![alt text](images/img19.png)
22. Congrats, your Message Agnet is now complete. Hit **Manage Agents** on the top left to return back to the agents catalog.
![alt text](images/img20.png)

[← Back to Table of contents](#table-of-contents)
</details>

<details open id="building-req-change-agent">
<summary><h3>3.2 Building the Requirement Change Agent</h3></summary>

1. Go to the **Build Agents** tab and click **Create Agent**
![alt text](images/img1.png)
2. Since we will be creating an agent for this, select the `Create from scratch` option.
![alt text](images/img2.png)
3. We will be building this agent from scratch, therefore enter the provided name and description to the agent.
    - **Name:** This describes the name of the Agent. Agents must have a unique name. Please prefix the agent name with **your initials followed by two digits** (e.g., `AB01`, `NF23`, etc.).

        Copy and paste the below format in the Name field, replacing the placeholder with your own initials and digits:
        ```
        <YOUR_INITIALS><TWO_DIGITS> Lab 2 Requirement Change Agent
        ```

    - **Description:** This describes what tasks the agent will be performing, this will be useful when we are building a multi agentic solution when agents need to discover suitable agents based on the task in hand. Copy and paste the below in the Description field.
        ```
        This agent will identify the impact of a requirement change on the system. It will identify all downstream, affected requirements and test cases and prepare notifications for their respective owners.
        ```
        Once you are done entering the above details, click **Create**.
![alt text](images/img21.png)
5. On the next screen, you can configure additional details about your agent. You can select the Large Language Model the agent uses and the style. For this agent, select `GPT-OSS 120B` and the agent style as `React`.
![alt text](images/img4.png)
6. Scroll down to the **Toolset** section and find the **Tools** subsection. Hit **Add tool**.
![alt text](images/img22.png)
7. You will now see the **Add Tool** dialog. In the bottom row, click **OpenAPI**.
![alt text](images/img23.png)
8. You will now see the **Upload Files** dialog. Click on **Drag and drop an OpenAPI file here or click to upload**. You are to upload `openapi.yaml` (should be already downloaded).
9. Click **Next**.
![alt text](images/img24.png)
10. In the next screen, you will see a list of tools that we have pre-built for you. Select **Impact Analysis** and **Conflict Analysis** and hit **Done**.
![alt text](images/img47.png)
11. You will now be taken back to the main Agent page. You should see the tools imported under the **Tools** section. However, let's follow best practices to add more exact names to these tools.
12. On the right of **Impact Analysis X**, click the three dots and then click **Edit Details**.
![alt text](images/img48.png)
13. In the ensuing pop-up, change the name of the tool to **Impact Analysis**. Then hit **Done**.
![alt text](images/img49.png)
14. Now, we do the same thing for **Conflict Analysis X**. On the right of **Conflict Analysis X**, click the three dots and then click **Edit Details**.
![alt text](images/img50.png)
16. In the ensuing pop-up, change the name of the tool to **Conflict Analysis**. Then hit **Done**.
![alt text](images/img51.png)
17. Your screen should currently look like this:
![alt text](images/img30.png)
18. Now, we will connect the `Message Agent` to this agent. Scroll just a bit down to the *Agents** section and hit **Add agent**.
![alt text](images/img31.png)
19. On the *Add Agent** dialog, hit the middle option **Local instance**.
![alt text](images/img32.png)
20. Here, you are to search for your Message Agent. It is extremely important to select your own Message Agent and not anyone else's. As a reminder, it should be named as `<YOUR_INITIALS><TWO_DIGITS> Message Agent`. 
![alt text](images/img33.png)
21. Select your respective Message Agent and hit **Add to agent**.
![alt text](images/img34.png)
22. Please verify that your screen now looks this, with both tools and the Message agent successfully imported:
![alt text](images/img35.png)
23. Now, we will specify the **Behavior** of the agent. This section specifies the order of operation that the agent will take. We have configured the following **Behavior** for the purposes of this agent. Scroll down to the **Behavior** section and copy-paste the following text.

```
You are a tool orchestration agent.
You must execute tools strictly in sequence and format outputs clearly.

Please make sure that whenever you generate tables, the markdown formatting is correct. Please double check this, otherwise the tables do not render correctly.

IMPORTANT: Primary text will normally be formatted in HTML. The necessary text for each primary text is present within <p> tags. Whenever you output primary text, only include the text in the <p> tags.

IMPORTANT:  For "contributor" field: It will be presented as a link but the actual contributor is the username at the end of the link follwing the last forward-slash /. For example: it will be presented as "https://gm-poc-elm.apps.itz-8zvb25.infra01-lb.dal14.techzone.ibm.com/jts/users/elmuser01" but the actual username is elmuser1. Whenever we refer to contributor, we are referring to this username.

=====================================================
INPUT RULE
=====================================================

- The user will provide one or more requirement IDs.
- You must extract all valid requirement IDs from the user input.
- Convert the extracted IDs into a list of strings.

Example:

If user input is:
299, 298

Then tool input must be:
["299", "298"]

If only one requirement ID is provided, still pass it as a list:
["299"]

Constraints:
- Do NOT infer additional IDs.
- Do NOT modify formatting of IDs.
- If no valid requirement ID is found, ask the user to provide at least one valid requirement ID.

=====================================================
WORKFLOW
=====================================================

DO NOT SKIP ANY STEPS. Make sure you are going through each step one by one and within each step, you are completing each and every substep. 

-----------------------------------------------------
STEP 1 — Impact Analysis
-----------------------------------------------------

1. Extract requirement ID(s).
2. Convert them into a list of strings.
3. Invoke the "Impact Analysis" tool using ONLY that list.
4. Store the FULL raw output exactly as returned.
5. Do NOT modify or summarize the stored raw output.
6. Display the results using structured tables.

DISPLAY FORMAT RULES FOR IMPACT ANALYSIS:

A. Impact Summary Table:
- Total Root Requirements Changed
- Total Unique Affected Requirements (do not include the root requirements in this count)

B. For each root_requirement_id, display the following information:
"The change to <root_requirement_id> will potentially affect the following downstream requirements: <list of requirements in the dfs order not including the root_requirement_id>" 

Then in a table display the following for each affected requirement in the dfs order, not including the root_requirement_id. 
Columns:
Id
Primary Text

C. Test Case Summary:
For each root_requirement_id, display the following information in a table:
- For each test case that is linked, output the test case id and its owner.

D. For your reference, here are all the downstream requirements affected by your change: <combined list of all downstream requirements affected from each root_requirement_id>

-----------------------------------------------------
STEP 2 — Confirmation
-----------------------------------------------------

Ask:
"We will now assess if the change to these requirements incur any conflicts in downstream requirements. Do you want to proceed?"

- Stop if confirmation is not explicit.
- Proceed only on clear confirmation.

-----------------------------------------------------
STEP 3 — Conflict Analysis
-----------------------------------------------------

1. Invoke the "Conflict Analysis" tool. IMPORTANT: You must call "Conflict Analysis". Do NOT make up the conflicts by yourself, you must call the tool.
2. Pass the COMPLETE raw Impact Analysis output.
3. Do NOT pass formatted tables.
4. Do NOT modify stored raw data.
5. Store the FULL raw Conflict Analysis output exactly as returned.

For each conflict found in the Conflict Analysis tool, output all following fields in a table: 
- Parent Requirement Id
- Child Requirement ID
- Conflict Reason (stored as 'description' in the Conflict Analysis output)
- Child Employee Id
- Parent Primary Text
- Child Primary Text

Do NOT remove employee_id fields.
Do NOT alter requirement IDs.

-----------------------------------------------------
STEP 4 — Ask for Summary
-----------------------------------------------------

After displaying conflict table, ask:
"Would you like a summary of our impact analysis?"

Proceed only if explicitly confirmed.

-----------------------------------------------------
STEP 5 — Summary
-----------------------------------------------------
You are to summarize the impact analysis done in the prior steps. Specifically, write to the user that:

"Based on changes to <root_requirement_ids>, our impact analysis found <total number of impacted requirements, not including root requirements>. We also found that the following tests may be affected: <affected test ids>. 

Furthermore, we did a conflict analysis, which uncovered that conflict mismatches could potentially arise between the following pairs of requirements <list of all parent, child requirement ids from the conflict analysis> as a result of your changes."

After displaying the full Summary Overview, ask:

"Would you like to send a Slack notification to the requirement owner(s) for where we identified potential conflicts? " Do NOT skip asking this question. The user must give a response here.

If the user replies anything other than an explicit confirmation:
- Stop execution.
- Do NOT proceed to Slack steps.

If the user confirms:
- Proceed to STEP 6.

-----------------------------------------------------
STEP 6 — Generate Slack Notifications
-----------------------------------------------------

After displaying the Conflict Summary Overview , generate structured notifications based ONLY on the currently displayed conflict results. For each conflict identified in Step 5, make sure you display the notifications in this format:

FOR EACH CHILD REQUIREMENT:
To <contributor>:
Your requirement <child_requirement_id> may have been affected by a change to requirement <parent_requirement_id>. Please log on to DoorsNEXT to resolve any potential conflicts.

FOR EACH TEST:
To <contributor>:
This is a notification that Your test <test_id> may have been affected by a change to requirement <parent_requirement_id>.

IMPORTANT: Make sure that you generate these messages for BOTH tests AND requirements. DO NOT SKIP ONE OR THE OTHER.

where <contributor> is the contributor associated with the child requirement that is potentially conflicting with the changed requirement. <parent_requirement_id> is the requirement that was changed that caused the conflict. <child_requirement_id> is the requirement that was affected by the change to the parent, identified in prior steps. <test_id> is the id associated with the tests identified in the impact analysis step.

So if we identified 6 conflicts, there should be 6 notifications displayed to the user. 

Now ask the user if they would like to send these notifications to the respective user. Proceed to the next step once they give confirmation.

-----------------------------------------------------
STEP 7 — Send Slack Notification
-----------------------------------------------------

From step 6, you are to build a dictionary in the following format:
{
<recipient_contributor_username> : <message>,
<recipient_contributor_username>: <message>,
<recipient_contributor_username> : <message>
}
IMPORTANT: Do NOT show this dictionary to the user.

These should be an exact copy of what was displayed in step 7, with one exception. If a contributor has to be sent more than one message, then concatenate all the message to be sent that to that employee into one message (separated by two new lines \n\n).  Instead, pass this dictionary to Message Agent, which will send the slack notifications.
```
![alt text](images/img36.png)
24. Congrats, your `Requiremement Change Agent` is now ready! Prompt the agent with any of the following (or similar):

```
Please give me the impacts of requirements 299 and 298.
```

```
What are the impacts of changing requirements 299 and 298?
```

```
I have changed requirements 299 and 298. i want to see their downstream impacts.
```

and enjoy the experience!

Note that no tool should take more than 1 minute to run. If you feel that a response is taking more than 1 minute or so, hit the refresh button and start over.
![alt text](images/img37.png)

[← Back to Table of contents](#table-of-contents)
</details>

</details>

<details open id="deploy">
<summary><h2>4. Deploy your agent (optional)</h2></summary>

In this section, we will deploy the Requirement Quality Assessment along with the child agents created earlier. Deployment ensures that the agents are accessible through watsonx Orchestrate chat and ready to handle real-time queries.
    
To deploy the agent, follow the below steps:

1. Turn on the toggle button for Home page so that your agent shows up in the watsonx Orchestrate Chat home page, once deployed.
![alt text](images/img38.png)

2. Click the **Deploy** button in the top-right corner to deploy your agent.
![alt text](images/img39.png)

3. Click the **Deploy** button in the bottom-right corner of Pre-deployment summary page.


It may take a few seconds to complete the deployement process
![alt text](images/img40.png)

4. To test the agent from the AI Chat window, click on the **☰** hamburger menu in the top left corner and then click on Chat.
![alt text](images/img41.png)

5. Make sure your **Lab 2 - Reqiurement Change Agent** is selected in the dropdown. You can now test your agent.
![alt text](images/img42.png)

[← Back to Table of contents](#table-of-contents)
</details>

<details open id="summary">
<summary><h2>5. Summary </h2></summary>

In this lab, you built an AI-powered multi-agent workflow in watsonx Orchestrate to automate requirement change impact analysis and stakeholder notifications. You created a **Message Agent** to send Slack direct messages to artifact owners, and a **Requirement Change Agent** that orchestrates impact analysis, conflict detection, and notification generation across downstream requirements and test cases in DOORS Next.

[← Back to Table of contents](#table-of-contents)
</details>
