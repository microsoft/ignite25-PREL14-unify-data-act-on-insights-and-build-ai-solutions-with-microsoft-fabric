@lab.Title

## Welcome to Your Lab Environment

**Interacting with your lab environment**
- Text formatted as an +++example+++ represents type text. Clicking on this text will automatically insert it to prevent any typing errors.
- Text formatted as an ++example++ represents copy text. Clicking on this text will copy it to your clipboard and can then be pasted to prevent any typing errors.
- Text formatted as an [example](https://www.microsoft.com/en-us/microsoft-fabric/getting-started) is a hyperlink that will open in a new browser tab or window.

**Getting started**

To begin, log into the virtual machine using the following Windows credentials:
- Username: +++@lab.VirtualMachine(Win11-Pro-Base).Username+++
- Password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++

Your Enter ID credentials, shown below, will be on the resources tab for reference throughout the lab:
- Entra username: +++@lab.CloudPortalCredential(User1).Username+++
- Entra password +++@lab.CloudPortalCredential(User1).Password+++

+++@lab.CloudPortalCredential(User1).TAP+++
+++@lab.CloudPortalCredential(User1).FabricCapacity+++
+++@lab.CloudResourceTemplate(FabricPerUserPOCv3).Parameters[capacityName]+++
@lab.CtrlAltDelete
@lab.CloudPortalCredential(User1).AccessToken

# End-to-End Data Engineering: <br> Modern Data Warehousing on Microsoft Fabric

## Lab 0 - Lab environment setup

Before you begin:

- Make sure you have read the overview on the [workshop homepage](<../README.md>).

This lab will cover:

- <a href="#0.1">Creating a workspace</a>
- <a href="#0.2">Creating a data warehouse</a>

<hr>

<h3 id="0.1">0.1 - Creating a workspace</h3>

1. Open your web browser and sign in to [Microsoft Fabric](https://fabric.microsoft.com).

1. From the left navigation bar, select **Workspaces** then select **+ New workspace**.

    *Note: The icons on the navigation bar can be pinned and unpinned. Therefore, the icons you see may differ from the screenshot.*

    ![](../assets/images/00_navigation_bar_new_workspace.png)

1. In the **Create a workspace** pane, in the **Name** box, enter **Modern Data Warehousing on Microsoft Fabric** (add some trailing characters if the name isn't available).

    ![](../assets/images/00_workspace_name.png)

1. Expand the **Advanced** section.
    - Change the **License mode** radio button to **Fabric capacity**, **Trial**, or **Premium capacity** based on the type of capacity you have available. *Note: Pro and Premium per user will not work for this workshop as they do not allow Fabric items to be created.*
    - From the **Capacity** dropdown list, select your capacity.
    - Select **Apply** to create and open the new workspace.

    ![](../assets/images/00_workspace_advanced.png)

<h3 id="0.2">0.2 - Creating a data warehouse</h3>

1. With the empty Fabric workspace still open, select **New item** located just below the workspace name.

    ![](../assets/images/00_new_item.png)

1. From the **All items** view, locate the **Store data** section and select the **Warehouse** tile.

    ![](../assets/images/00_new_item_list.png)

1. On the **New warehouse** dialog box, enter the name **WideWorldImportersDW** and select **Create**. The warehouse will be created and open to the *Build a warehouse* landing page with a set of links to get started quickly. 

    ![](../assets/images/00_new_warehouse_landing_page.png)

## Next steps
In this lab you configured your environment which includes a workspace, a capacity, and a data warehouse. This is the foundational infrastructure for the remaining labs in this workshop. You are now ready to get started learning how to build a modern data warehouse on Microsoft Fabric!

- Continue to [Lab 1 - Getting started](<01 - Getting started.md>)
- Return to the [workshop homepage](<../README.md>)

## Additional Resources
- [Getting started with Microsoft Fabric](https://www.microsoft.com/en-us/microsoft-fabric/getting-started)
- [Enable Microsoft Fabric for your organization](https://learn.microsoft.com/en-us/fabric/admin/fabric-switch)
- [Fabric region availability](https://learn.microsoft.com/en-us/fabric/admin/region-availability)
- [Microsoft Fabric concepts and licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses)
- [Microsoft Fabric trial capacity](https://learn.microsoft.com/en-us/fabric/fundamentals/fabric-trial)
- [Create a Warehouse in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/create-warehouse)
