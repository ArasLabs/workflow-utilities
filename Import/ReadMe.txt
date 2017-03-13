Author:
-------
Rolf Laudenbach (rlaudenbach@aras.com)

Version:
--------
v1-4  (Nov 2012)
  - added "Clone WorkflowActivityToPending" method connectd to onActivate event of "Draw Changes" activiy of 
    "Express Team DCO" workflow. When looping back from "Team Review" to "Draw Changes" the "Team Review" activity
    gets cloned and set to status "Pending" so new assignments can be added manually (or with logic).
    This overrides standard behavior where "Team Activity would be cloned only when visited again.
    But then the status would remain "Closed" (not updateable)


Description (Workflow Utilities)
--------------------------------

Import 1 (import as super user 'root')
======================================
- adds property 'place_holder_resolution_rule' to ItemType "Identity" and to "Form" "Itentity Admin Form"
- adds Method 'WF ResolvePlaceHolderOnActivity'
    this server method does NOT get attached anywhere.
    It contains logic to do more dynamic assignments on workflow activities.
    It must be used on an activity's Server Event "onAssign"
    To to learn more open the method code and read the comments.


Import 2 (import as 'admin')
=============================
- adds a Workflow icon to the toolbar of the "SignOffs" tab
- Add server methods that change the owned_by_id of all unclosed activities of a started Workflow Process.
  This logic can be either triggered from a server event a workflow transition or 
  a server event on a life cycle transition.
  The owner of a workflow controlled item will then be the owner of all activities, as well,
  allowing to add or remove existing assignements on the workflow process "adhoc"

- adds sample trigger of "CopyChangeOwnerToWorkflow" on life cycle transition -> "In Work" on "Express DCO and ECO"


(Optional) Import 3 (import as 'admin')
======================================
- replaces standard workflow "Express DCO" with "Express Team DCO" on item Type Document.


Dependencies:
-------------
None


available as Community Project
------------------------------
No


available separetely on Partner Portal
--------------------------------------
No


Version History:
--------
v1-3  (Sep 2012)
  - added warning popup to Report "Workflow Process History". if WF item is locked, voting does not start.

v1-2  (Apr 2012)


Notice of Liability
-------------------
The information contained in this document and the import packages are distributed on an "As Is" basis, 
without warranty of any kind, express or implied, including, but not limited to, the implied warranties 
of merchantability and fitness for a particular purpose or a warranty of non-infringement. Aras shall have 
no liability to any person or entity with respect to any loss or damage caused or alleged to be caused 
directly or indirectly by the information contained in this document or by the software or hardware products 
described herein.

  