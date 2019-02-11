---
id: 762
title: PeopleSoft HTML Element IDs
date: 2016-11-14T05:21:49+00:00
guid: https://www.peoplesoftmods.com/?p=762
permalink: /tips-and-tricks/peoplesoft-html-element-ids/
categories:
  - Tips and Tricks
---
The HTML element IDs that appear on PeopleSoft pages follow an officially undocumented naming convention.  It would be nice to know the exact naming convention that is being used on these HTML element IDs so that there would be no uncertainty when it comes to DOM manipulation with injected client side code.  Fortunately, there have been many PeopleSoft experts in the past that have demonstrated how the HTML element IDs on PeopleSoft pages typically have their record and field names present in the ID.  It is worth noting that not all fields follow the RECORDNAME_FIELDNAME naming convention.  This is true for fields that have a value set for the page field name in the page field properties in App Designer.  An example of this would be the National ID field that appears on the relationships page.

<!--more-->

[<img class="alignnone size-full wp-image-763" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page.png" alt="Relationships_Page" width="1277" height="532" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page.png 1277w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page-300x125.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page-768x320.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page-1024x427.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page-912x380.png 912w" sizes="(max-width: 1277px) 100vw, 1277px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Relationships_Page.png)

Inspecting the HTML element of this field shows an ID of SA\_MASK\_NID2.

[<img class="alignnone size-full wp-image-764" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element.png" alt="National_ID_Element" width="837" height="433" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element.png 837w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element-300x155.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element-768x397.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element-735x380.png 735w" sizes="(max-width: 837px) 100vw, 837px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/11/National_ID_Element.png)

However, the actual field name is NATIONAL\_ID and the record name is PERS\_NID\_PRM\_VW.

[<img class="alignnone size-full wp-image-765" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties.png" alt="Page_Field_Properties" width="1046" height="490" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties.png 1046w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties-300x141.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties-768x360.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties-1024x480.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties-811x380.png 811w" sizes="(max-width: 1046px) 100vw, 1046px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Properties.png)

The SA\_MASK\_NID2 value that is used as the element ID for this field is specified under the general tab in the page field properties as the page field name.

[<img class="alignnone size-full wp-image-766" src="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name.png" alt="Page_Field_Name" width="1029" height="496" srcset="https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name.png 1029w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name-300x145.png 300w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name-768x370.png 768w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name-1024x494.png 1024w, https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name-788x380.png 788w" sizes="(max-width: 1029px) 100vw, 1029px" />](https://www.peoplesoftmods.com/wp-content/uploads/2016/11/Page_Field_Name.png)

So it seems that if a field has a value for the page field name, then PeopleSoft uses this value as the HTML element ID, otherwise the record and field name will be used.  If this assumption is true, then the PeopleSoft meta-data tables can be used to determine what the HTML element ID will be for any given field name.  Below is a query that can be used for this purpose.

<pre>SELECT DISTINCT TB2.PNLGRPNAME AS COMPONENT 
 , TB1.PNLNAME AS PAGE 
 , TB1.RECNAME AS RECORD
 , TB1.FIELDNAME AS FIELD 
 , NVL(TRIM(TB1.PNLFIELDNAME), TRIM(TB1.RECNAME || '_' || TB1.FIELDNAME)) AS ELEMENT_ID 
 FROM PSPNLFIELD TB1, PSPNLGROUP TB2 
 WHERE TB1.PNLNAME = TB2.PNLNAME
 AND TB1.FIELDNAME = 'NATIONAL_ID' -- put field name here
 --AND TB1.RECNAME = 'PERS_NID_PRM_VW' -- put record name here
 --AND TB1.PNLNAME = 'RELATIONSHIPS' -- put page name here
 --AND TB2.PNLGRPNAME = 'RELATIONSHIPS' -- put component name here
 UNION 
 SELECT DISTINCT TB3.PNLGRPNAME AS COMPONENT 
 , TB2.PNLNAME AS PAGE 
 , TB1.RECNAME AS RECORD
 , TB1.FIELDNAME AS FIELD 
 , NVL(TRIM(TB1.PNLFIELDNAME), TRIM(TB1.RECNAME || '_' || TB1.FIELDNAME)) AS ELEMENT_ID 
 FROM PSPNLFIELD TB1, PSPNLFIELD TB2 , PSPNLGROUP TB3 
 WHERE TB1.PNLNAME = TB2.SUBPNLNAME 
 AND TB2.FIELDTYPE = '11'
 AND TB2.PNLNAME = TB3.PNLNAME 
 AND TB1.FIELDNAME = 'NATIONAL_ID' -- put field name here
 --AND TB1.RECNAME = 'PERS_NID_PRM_VW' -- put record name here
 --AND TB2.PNLNAME = 'RELATIONSHIPS' -- put page name here
 --AND TB3.PNLGRPNAME = 'RELATIONSHIPS' -- put component name here</pre>

This query plays a big role in the configurable interface of my [field-level data masking solution](https://youtu.be/RJUSCyy1rKg). Hopefully the PeopleSoft HTML element ID naming convention stays consistent in the future.