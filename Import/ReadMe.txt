Notice of Liability
-------------------
The information contained in this document and the import packages are distributed on an "As Is" basis, 
without warranty of any kind, express or implied, including, but not limited to, the implied warranties 
of merchantability and fitness for a particular purpose or a warranty of non-infringement. Aras shall have 
no liability to any person or entity with respect to any loss or damage caused or alleged to be caused 
directly or indirectly by the information contained in this document or by the software or hardware products 
described herein.



Author:
-------
Rolf Laudenbach (rlaudenbach@aras.com)


Version:
--------
v2-0  (Apr 2016) - upgraded to work with Aras Innovator 10 & 11
  - converted VB Methods to C#.

  - improved identity replacement rule syntax to use AML.
  - updated method "WF ResolvePlaceHolderOnActivity" to read AML like resolution rule. Added support for new rule option "propertyToMatchActivityName".


Description (Workflow Utilities)
--------------------------------
A mix of various features (primarily methods) to help automate workflows. Importing the package does not necessarily activate these
features. Please read the documentation to learn how to best make use of them.

- Resolve Placeholder Indentities on WF Activities (onAssign). (see "Feature Details" the end of this ReadMe)
  this may be the most powerful feature in this package. It consists of:
	 * a method: "WF ResolvePlaceHolderOnActivity" to be added to Activity Server Events.
	 * And a new property on Identities "place_holder_resolution_rule" 

- Adds a toolbar button on Workflow History Form to open the view of the Workflow Process directly from the “SignOffs” tab

- Logic to change WF Activities’ permissions to allow “adhoc” Workflow Assignments.
	Can be triggered from Life Cycle transitions or when Workflow Activities get activated.
	An example is added to the Express DCO’s life cycle.

- Logic to set activities to “automated” ,if no assignments configured. 
	This way you can set up many sequential activities with no assignments and use “ad-hoc” assignment
	on the ones you want to activate after the workflow process was started.

- Logic to „clone“ an activity when you loop to it again so you can still add different „ad-hoc“ assignments.
	The „Team Review Express DCO“ workflow (sample) has an example how to use this logic.

- Adds a “single Review Express DCO Workflow” that can be used instead of the standard workflow.

- Adds a “Team Review Express DCO Workflow” that can be used instead of the standard workflow.

- Sample code to show how to save data from Workflow activities to the item connected to the workflow process.



Installation Steps
------------------
use package import utility to import packages in this sequence. (logon as "admin" or "root" as indicated and use option "merge" for all steps)

------------------
	1_core-extensions - import as "root"

	2_placeholder-resolution - import as "root"
		- adds Method 'WF ResolvePlaceHolderOnActivity'
		  this server method does NOT get attached anywhere.

	3_wf_utilities - import as "admin"
		NOTE: copy the codeTreeOverlay files to your Aras installation folder

		- Adds server methods that change the owned_by_id of all unclosed activities of a started Workflow Process.
		  This logic can be either triggered from a server event a workflow transition or 
		  a server event on a life cycle transition.
		  The owner of a workflow controlled item will then be the owner of all activities, as well,
		  allowing to add or remove existing assignements on the workflow process "adhoc"


	(Optional) 4_add CopyOwner to DCO LCs
		- Adds sample trigger of "CopyCtrldItemOwnerToWorkflow" on life cycle transition -> "In Work" on "Express DCO and ECO"

	(Optional) 5_add TeamReview WF to DCO
		replaces standard workflow "Express DCO" with "Express Team DCO" on item Type Document.



Dependencies:
-------------
None


available as Community Project
------------------------------
Yes


available separetely on Partner Portal
--------------------------------------
No


Version History:
--------
v1-4  (Nov 2012)
  - added "Clone WorkflowActivityToPending" method connectd to onActivate event of "Draw Changes" activiy of 
    "Express Team DCO" workflow. When looping back from "Team Review" to "Draw Changes" the "Team Review" activity
    gets cloned and set to status "Pending" so new assignments can be added manually (or with logic).
    This overrides standard behavior where "Team Activity would be cloned only when visited again.
    But then the status would remain "Closed" (not updateable)

v1-3  (Sep 2012)
  - added warning popup to Report "Workflow Process History". if WF item is locked, voting does not start.

v1-2  (Apr 2012)


---------------
Feature Details
---------------

The "place_holder_resolution_rule"

/*
  This logic will look at the list of assignees of the activity this method is connected to (via "opnAssign" event).
  There it will look for identitity names with a special naming (format). ${<identity_name>}
  Any identity starting with "${"  it will treat as a place holder and try to resolve it.

  The rules how to resolve are expected in the identity’s property "place_holder_resolution_rule" (lentgth = 512).
  The result of the rules resolution is one or more identities to be used instead of the place holder.

  If an identity is already listed, but would also be added again by a resolved identity the following precedence rules apply:
     - no duplicate identities will be added !
     - if the existing identity has flag "required" set,  it stays
     - if the existing identity has flag "required" NOT set and the resolved identity has the flag set, the flag will be set on the existing,
     - voting weight, for_all_members_flag, escalate_to settings of exisitng identities will always stay
     - voting weight setting on place holder identity will be used for resolved identity
     - for_all_members_flag, escalate_to settings on place holder will be ignored for resolved identity
 
  The placeholder identity will be removed from the Activity and the new 'resolved' identities will be added.

  These Rules define where to find a property of type "Item" with Data Source "Identity". 
  An AML syntax is used to define a query started from the workflow controlled item. Tag id="{@id}" must be used on the top <Item ...> attributes.

  The <Item ...> where the identity property is must be tagged with attribute: resolveToIdentity="<prop_name>" 
  (Where <prop_name> = name of property pointing to an identity to be use in this resolution).

  Optionally the same <Item ...> with the identity property can be tagged with attribute propertyToMatchActivityName="<act_prop_name>" 
  (Where <act_prop_name> = name of property containing the name of the workflow activity the identity shall be added to).

  Here are some examples:
*/

/// Case 1 - Example: "Simple ECO" has a property "management_CRB_id" pointing to an identity that shall be used instead of placeholder identity
//  placeholder identity name => ${ECO Mgmt CRB} 
//  place_holder_resolution_rule => <Item type="Simple ECO" id="{@id}" resolveToIdentity="management_CRB_id"></Item>

/// Case 2 - Example: "PR" has a property pointing to an item with a property "owned_by_id" pointing to an identity that shall be used instead of placeholder identity
//  placeholder identity name => ${PR Affected Item Owner} 
//  place_holder_resolution_rule => 
//			<Item type="PR" id="{@id}" >
//			 <affected_item>
//			  <Item type="Change Controlled Item" resolveToIdentity="owned_by_id" />
//			 </affected_item>
//			</Item>

/// Case 3 - Example: "Simple ECO" has a NULL relationship "Simple ECO Approvers" with a property "approver_id" pointing to an identity.
///                     Those found for all relationships shall be used instead of placeholder identity.
//  placeholder identity name => ${Simple ECO Approvers} 
//  place_holder_resolution_rule => 
//			"<Item type="Simple ECO" id="{@id}" >
//			<Relationships>
//			  <Item type="Simple ECO Approvers" resolveToIdentity="approver_id" />
//			</Relationships>
//			</Item>

/// Case 3+ - Example: "Simple ECO" has a relationship "Simple ECO Approver" pointing to an identity (via standard property related_id)
///			The relationship also has a property that identifies the name of the WF activity the related identity shall be connected to.
///        		Those found for all relationships shall be used instead of placeholder identity.
//  placeholder identity name => ${Simple ECO Approvers by Activity}
//  place_holder_resolution_rule => 
//			"<Item type="Simple ECO" id="{@id}" >
//			<Relationships>
//			  <Item type="Simple ECO Approver" resolveToIdentity="related_id" propertyToMatchActivityName='wf_activity_name' />
//			</Relationships>
//			</Item>

/// Case 4 - Example: "ECN" has a relationship "ECN Affected Item" with related items of type "Affected Item" that have a property "new_item_id" pointing to an item of type
///			"Change Controlled Item" with a property "owned_by_id" pointing to an identity. 
///			All "Change Controlled Item" owners shall be used instead of placeholder identity.
//  placeholder identity name => ${ECN AffItem NewItem Owners} 
//  place_holder_resolution_rule => 
//			<Item type="ECN" id="{@id}" >
//			<Relationships>
//			 <Item type="ECN Affected Item">
//			  <related_id>
//			   <Item type="Affected Item" >
//			    <new_item_id>
//			     <Item type="Change Controlled Item" resolveToIdentity="owned_by_id" />
//			    </new_item_id>
//			   </Item>
//			  </related_id>
//			 </Item>
//			</Relationships>
//			</Item>
