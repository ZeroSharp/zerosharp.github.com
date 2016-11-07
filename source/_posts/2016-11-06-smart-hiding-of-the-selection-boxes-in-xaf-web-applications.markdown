---
layout: post
title: "Smart hiding of the selection boxes in XAF web applications"
date: 2016-11-06 18:39
comments: true
categories: [c#, devexpress, xaf]
description: How to hide the selection box in expressAppFramework web applications when there are no selection-dependent actions available.
---
When an [XAF](https://www.devexpress.com/products/net/application_framework/) list view has no selection-based actions available, the selection box still appears in the grid. Users get confused. In this post, we'll look at a workaround.

## The problem ##

In the XAF MainDemo, lets make Departments read-only for the User role.

```c# Updater.cs
userRole.AddTypePermissionsRecursively<Department>(SecurityOperations.Create, SecurityPermissionState.Deny);
userRole.AddTypePermissionsRecursively<Department>(SecurityOperations.Write, SecurityPermissionState.Deny);
userRole.AddTypePermissionsRecursively<Department>(SecurityOperations.Delete, SecurityPermissionState.Deny);
```

Then start the web application, login as John and navigate to the Departments list view. There is a column selection box, but it serves no purpose. There are no actions that depend on a grid selection.

{% imgcap /images/blog/selection-visibility-controller-001.png Without the SelectionColumnVisibilityController %}

## The fix ##

Here is a controller which calculates whether there are any available actions which require one or more rows to be selected. If there are none, the selection box will not appear.

Add the following controller to the MainDemo.Module.Web project. It hides the selection box if there are no actions which depend on a grid selection.

```c# SelectionColumnVisibilityController.cs
using System;
using DevExpress.ExpressApp;
using DevExpress.ExpressApp.Actions;
using DevExpress.ExpressApp.Editors;
using DevExpress.ExpressApp.SystemModule;
using DevExpress.Web;
using System.Linq;

namespace MainDemo.Module.Web.Controllers
{
    public class SelectionColumnVisibilityController : ViewController
    {
        public SelectionColumnVisibilityController()
        {
            TargetViewType = ViewType.ListView;
        }

        private bool IsSelectionColumnVisible()
        {
            bool isSelectionColumnRequired = false;
            // remove checkbox if there are no available actions
            foreach (Controller controller in Frame.Controllers)
            {
                if (!controller.Active)
                    continue;

                if (controller.Actions.Count == 0)
                    continue;

                bool allowEdit = true;
                if ((Frame is NestedFrame) && (((NestedFrame)Frame).ViewItem is PropertyEditor))
                    allowEdit = (bool)((PropertyEditor)((NestedFrame)Frame).ViewItem).AllowEdit;

                foreach (ActionBase action in controller.Actions)
                {
                    if (action.SelectionDependencyType == SelectionDependencyType.RequireMultipleObjects)
                    {
                        if (action.Active || IsActionInactiveBySelectionContext(action))
                        {
                            if (action.Enabled || IsActionDisabledBySelectionContext(action))
                            {
                                isSelectionColumnRequired = true;
                                break;
                            }
                        }
                    }
                }
                if (isSelectionColumnRequired)
                    break;
            }
            return isSelectionColumnRequired;
        }

        private bool IsActionInactiveBySelectionContext(ActionBase action)
        {
            if (action.Active)
                return true;
            else
            {
                foreach (string item in action.Active.GetKeys())
                {
                    if (item == ActionBase.RequireMultipleObjectsContext || item == ActionBase.RequireSingleObjectContext)
                        continue;
                    if (!action.Active[item])
                        return false;
                }
                return true;
            }
        }

        private bool IsActionDisabledBySelectionContext(ActionBase action)
        {
            if (action.Enabled)
                return true;
            else
            {
                foreach (string item in action.Enabled.GetKeys())
                {
                    if (item == ActionBase.RequireMultipleObjectsContext ||
                        item == ActionBase.RequireSingleObjectContext ||
                        item == ActionsCriteriaViewController.EnabledByCriteriaKey)
                        continue;
                    if (!action.Enabled[item])
                        return false;
                }
                return true;
            }
        }

        protected override void OnViewControlsCreated()
        {
            base.OnViewControlsCreated();
            ASPxGridView grid = ((ListView)this.View).Editor.Control as ASPxGridView;
            if (grid != null)
            {
                grid.Load += grid_Load;
                grid.DataBound += grid_DataBound;
            }
        }

        protected override void OnDeactivated()
        {
            base.OnDeactivated();
            ASPxGridView grid = ((ListView)this.View).Editor.Control as ASPxGridView;
            if (grid != null)
            {
                grid.DataBound -= grid_DataBound;
                grid.Load -= grid_Load;
            }
        }

        void grid_Load(object sender, EventArgs e)
        {
            SetSelectionColumnVisibility(sender, e);
        }

        void grid_DataBound(object sender, EventArgs e)
        {
            SetSelectionColumnVisibility(sender, e);
        }

        private void SetSelectionColumnVisibility(object sender, EventArgs e)
        {
            bool isSelectionColumnVisible = IsSelectionColumnVisible();
            if (!isSelectionColumnVisible)
            {
                var grid = (ASPxGridView)sender;
                var selectionBoxColumn =
                    grid.Columns
                        .OfType<GridViewCommandColumn>()
                        .Where(x => x.ShowSelectCheckbox)
                        .FirstOrDefault();

                if (selectionBoxColumn != null)
                {
                    selectionBoxColumn.Visible = false;
                }
            }
        }
    }
}
```

Run the application again and see the difference. Now the grid looks like this. Notice, there is no longer a selection box on the row.

{% img /images/blog/selection-visibility-controller-002.png With the SelectionColumnVisibilityController%}

By the way, this is how it looks with old-style XAF web apps.

{% imgcap /images/blog/selection-visibility-controller-003.png Without the SelectionColumnVisibilityController %}

{% imgcap /images/blog/selection-visibility-controller-004.png With the SelectionColumnVisibilityController %}

