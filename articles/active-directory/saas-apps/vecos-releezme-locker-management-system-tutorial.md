---
title: 'Tutorial: Azure AD SSO integration with VECOS Releezme Locker management system'
description: Learn how to configure single sign-on between Azure Active Directory and VECOS Releezme Locker management system.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/20/2021
ms.author: jeedes

---

# Tutorial: Azure AD SSO integration with VECOS Releezme Locker management system

In this tutorial, you'll learn how to integrate VECOS Releezme Locker management system with Azure Active Directory (Azure AD). When you integrate VECOS Releezme Locker management system with Azure AD, you can:

* Control in Azure AD who has access to VECOS Releezme Locker management system. Access to the VECOS Releezme Locker Management System is only needed for users who need to manage the lockers, i.e., facility managers, service desk employees, etc.
* Enable your users to be automatically signed-in to VECOS Releezme Locker management system with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* VECOS Releezme Locker management system single sign-on (SSO) enabled subscription.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* VECOS Releezme Locker management system supports **SP** initiated SSO.

> [!NOTE]
> Identifier of this application is a fixed string value so only one instance can be configured in one tenant.

## Add VECOS Releezme Locker management system from the gallery

To configure the integration of VECOS Releezme Locker management system into Azure AD, you need to add VECOS Releezme Locker management system from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **VECOS Releezme Locker management system** in the search box.
1. Select **VECOS Releezme Locker management system** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

## Configure and test Azure AD SSO for VECOS Releezme Locker management system

Configure and test Azure AD SSO with VECOS Releezme Locker management system using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in VECOS Releezme Locker management system.

To configure and test Azure AD SSO with VECOS Releezme Locker management system, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure VECOS Releezme Locker management system SSO](#configure-vecos-releezme-locker-management-system-sso)** - to configure the single sign-on settings on application side.
    1. **[Create VECOS Releezme Locker management system test user](#create-vecos-releezme-locker-management-system-test-user)** - to have a counterpart of B.Simon in VECOS Releezme Locker management system that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **VECOS Releezme Locker management system** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

   ![Edit Basic SAML Configuration](common/edit-urls.png)

1. On the **Basic SAML Configuration** section, perform the following steps: 

	a. In the **Identifier(Entity ID)** text box, type a URL using the following pattern: 
    `https://<baseURL>/`

    b. In the **Reply URL** textbox, type a URL using the following pattern: 
    `https://<baseURL>/Saml2/Acs`
    
    c. In the **Sign on URL** text box, type a URL using the following pattern:   
    `https://<baseURL>/sso` (optionally add the `?companycode=` query parameter with the company code value given by VECOS.)

    > [!NOTE]
    > These values are not real. Update these values with the actual Identifier, Reply URL and Sign on URL. Contact [VECOS Releezme Locker management system support team](mailto:servicedesk@vecos.com) what region you are connecting to. Depending on your region, the URL's below will be different:

    | **Region** | **baseURL** |
    |-------|-------|
	| Europe| `https://www.releezme.net` |
	| North America | `https://na.releezme.net` |
	| Asia Pacific | `https://au.releezme.net` |

1. On the **Set up single sign-on with SAML** page, In the **SAML Signing Certificate** section, click copy button to copy **App Federation Metadata Url** and save it on your computer.

	![The Certificate download link](common/copy-metadataurl.png)

## Configure VECOS Releezme Locker management system Roles

1. In the Azure portal, select **App Registrations**, and then select **All applications**.
1. In the app registrations list, select **VECOS Releezme Locker management system**.
1. In the app registration open **App roles**.
1. In the app roles page, create a new app role by clicking **Create app role**
1. In the **display name** field enter a name for the role, e.g., `VECOS Company Facility Manager`.
1. Select **Users/Groups** as the **Allowed member types** value.
1. Enter the VECOS Releezme Locker management system role name in the **Value** field. See table below.
1. Click **Apply**.

| Role | Role Value | Description |
| -- | --------- | ---------- |
| Service Desk | CompanyServiceDesk | Limited access service desk. Mostly read-only access |
| Service Desk+ | CompanyServiceDeskPlus | Advanced version of the service desk with more read/write access |
| Facility Manager | CompanyFacilityManager | Facility Manager with access to setup of the company |
| Facility Manager+ | CompanyFacilityManagerPlus | Advanced Facility Manager with additional access within the company. |
| Administrator | CompanyAdmin | Administrator with full company access |

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to VECOS Releezme Locker management system.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **VECOS Releezme Locker management system**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure VECOS Releezme Locker management system SSO

To configure single sign-on on **VECOS Releezme Locker management system** side, you need to send the **App Federation Metadata Url** to [VECOS Releezme Locker management system support team](mailto:servicedesk@vecos.com). They set this setting to have the SAML SSO connection set properly on both sides.

### Create VECOS Releezme Locker management system test user

In this section, you create a user called Britta Simon in VECOS Releezme Locker management system. Work with [VECOS Releezme Locker management system support team](mailto:servicedesk@vecos.com) to add the users in the VECOS Releezme Locker management system platform. Users must be created and activated before you use single sign-on.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration with following options. 

* Click on **Test this application** in Azure portal. This will redirect to VECOS Releezme Locker management system Sign-on URL where you can initiate the login flow. 

* Go to VECOS Releezme Locker management system Sign-on URL directly and initiate the login flow from there.

* You can use Microsoft My Apps. When you click the VECOS Releezme Locker management system tile in the My Apps, this will redirect to VECOS Releezme Locker management system Sign-on URL. For more information about the My Apps, see [Introduction to the My Apps](../user-help/my-apps-portal-end-user-access.md).

## Next steps

Once you configure VECOS Releezme Locker management system you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).