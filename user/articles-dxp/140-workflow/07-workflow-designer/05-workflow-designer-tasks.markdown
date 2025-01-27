---
header-id: creating-tasks-in-workflow-designer
---

# Creating Tasks in the Workflow Designer

[TOC levels=1-4]

Task nodes have several parts and are the most complex parts of a workflow
definition. Unlike other workflow nodes, task nodes have Assignments, because a
User is expected to *do something* (often approve or reject the submitted asset)
when a workflow process enters the task node: the assignment specifies who that
User is. 

Commonly, task nodes contain Notifications, Assignments, and Actions (defined in
scripts). See more about Notifications and Actions in the article on 
[workflow nodes](/docs/7-2/user/-/knowledge_base/u/workflow-definition-nodes). 
Task nodes and their assignments are more complex and deserve their own article
(this one).

To get started, drag and drop a task node on your workflow canvas if you haven't
already. Open its Properties and give it a name. Then double click *Actions* in
the task's Properties pane.

You can define a notification (often Task Assignee is appropriate), or write a
Groovy script defining an action that's triggered for your task.

Next learn about creating Assignments for your task nodes. 

## Assignments

Workflow tasks must be completed by a User. You can choose how you want to
configure your assignments. 

![Figure 1: You can add an Assignment to a Task node.](../../../images-dxp/workflow-designer-assignment.png)

You can choose to add assignments to specific Roles, multiple Roles of a Role
Type (Organization, Site, or regular Role types), to the Asset Creator, to
Resource Actions, or to specific Users. Additionally, you can write a script to
define the assignment.

Assigning tasks to Roles, Organizations, or Asset Creators is a straightforward
concept, but what does it mean to assign a workflow task to a Resource Action?
Imagine an *UPDATE* resource action. If your workflow definition specifies the
UPDATE action in an assignment, anyone who has permission to update the type of
asset being processed in the workflow is assigned to the task. You can configure
multiple assignments for a task.

### Resource Action Assignments

*Resource actions* are operations performed by Users on an application or
entity. For example, a User might have permission to update Message Boards
Messages. This is called an UPDATE resource action, because the User can update
the resource. If you're still uncertain about what resource actions are, refer
to the developer tutorial on the 
[permission system](/docs/7-2/tutorials/-/knowledge_base/t/defining-application-permissions)
for a more detailed explanation.

To find all the resource actions that have been configured, you need access to
the Roles Admin application in the Control Panel (in other words, you need
permission for the VIEW action on the Roles resource).

- Navigate to Control Panel &rarr; Users &rarr; Roles.
- Add a new Regular Role. See the 
  [article on managing roles](/docs/7-2/user/-/knowledge_base/u/roles-and-permissions)
  for more information.
- Once the Role is added, navigate to the Define Permissions interface for the
  Role.
- Find the resource whose action you want to use for defining your workflow
  assignment.

How do you go from finding the resource action to using it in the workflow? Use
the Workflow Designer's interface for setting up a resource action assignment.

When configuring your task node's Assignment, select Resource Actions as the
Assignment Type, then specify the Resource Actions to use for the assignment
(for example, UPDATE).

![Figure 2: Configure resource action assignments in the Workflow Designer.](../../../images-dxp/workflow-designer-resource-action-assignment.png)

Here's what the assignment looks like in the Source (Workflow XML) tab:

    <assignments>
        <resource-actions>
            <resource-action>UPDATE</resource-action>
        </resource-actions>
    </assignments>

As usual, assign the workflow to the appropriate workflow enabled asset.

Now when the workflow proceeds to the task with the resource action assignment,
Users with `UPDATE` permission on the resource (for example, Message Boards
Messages) are notified of the task and can assign it to themselves (if the
notification is set to Task Assignees). Specifically, Users see the tasks in
their *My Workflow Tasks* application under the tab *Assigned to My Roles*.

Use all upper case letters for resource action names. Here are some common
resource actions:

    UPDATE
    ADD
    DELETE
    VIEW
    PERMISSIONS
    SUBSCRIBE
    ADD_DISCUSSION

You can determine the probable resource action name from the permissions screen
for that resource. For example, in Message Boards, one of the permissions
displayed on that screen is *Add Discussion*. Convert that to all uppercase and
replace the space with an underscore, and you have the action name. 

### Scripted Assignments

You can also use a script to manage the assignment. Here's the script for the
Review task assignment in the Scripted Single Approver workflow definition
(`single-approver-definition-scripted-assignment.xml`):

    import com.liferay.portal.kernel.model.Group;
    import com.liferay.portal.kernel.model.Role;
    import com.liferay.portal.kernel.service.GroupLocalServiceUtil;
    import com.liferay.portal.kernel.service.RoleLocalServiceUtil;
    import com.liferay.portal.kernel.util.GetterUtil;
    import com.liferay.portal.kernel.workflow.WorkflowConstants;

    long companyId = GetterUtil.getLong((String)workflowContext.get(WorkflowConstants.CONTEXT_COMPANY_ID));

    long groupId = GetterUtil.getLong((String)workflowContext.get(WorkflowConstants.CONTEXT_GROUP_ID));

    Group group = GroupLocalServiceUtil.getGroup(groupId);

    roles = new ArrayList<Role>();

    Role adminRole = RoleLocalServiceUtil.getRole(companyId, "Administrator");

    roles.add(adminRole);

    if (group.isOrganization()) {
        Role role = RoleLocalServiceUtil.getRole(companyId, "Organization Content Reviewer");

        roles.add(role);
    }
    else {
        Role role = RoleLocalServiceUtil.getRole(companyId, "Site Content Reviewer");

        roles.add(role);
    }

    user = null;
						
Don't let all that code intimidate you. It's just assigning the task to the
*Administrator* Role, then checking whether the *group* of the asset is an
Organization and assigning it to the *Organization Content Reviewer* Role if it
is. If it's not, it's assigning the task to the *Site Content Reviewer* Role.

Note the `roles = new ArrayList<Role>();` line above. In a scripted assignment,
the `roles` variable is where you specify any Roles the task is assigned to. For
example, when `roles.add(adminRole);` is called, the Administrator role is added
to the assignment.

## Related Topics [](id=related-topics)

[Kaleo Forms](/docs/7-2/user/-/knowledge_base/u/kaleo-forms)

[Activating Workflow](/docs/7-2/user/-/knowledge_base/u/activating-workflow)

[Liferay's Workflow Framework](/docs/7-2/frameworks/-/knowledge_base/f/the-workflow-framework)

[Dynamic Data Lists](/docs/7-2/user/-/knowledge_base/u/dynamic-data-lists) 

