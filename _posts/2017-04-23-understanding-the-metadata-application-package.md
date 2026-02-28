---
id: 974
title: 'Understanding the %metadata Application Package'
date: 2017-04-23T16:17:18+00:00
guid: https://www.peoplesoftmods.com/?p=974
permalink: /tips-and-tricks/understanding-the-metadata-application-package/
categories:
  - Tips and Tricks
---
The %metadata application package in PeopleSoft is very intriguing.  This app package is unlike any other as we (PeopleSoft Developers) do not have access to the implementation of this package.  When you try to open this package in App Designer, the IDE acts as if the package does not exist.  However, if you correctly reference this package’s contents (sub-packages, classes, methods, properties, etc.) then App Designer does not bat an eye.  It should be well understood that (our) usage of this package is not supported by Oracle as it is undocumented.  However, there are currently no measures in place to prevent blind usage this app package.  While I definitely do not advise the usage of this package in any legitimate PeopleSoft system, I thought it would be a fun educational exercise to try to understand this mysterious app package.

<!--more-->

I would like to start this off by making it clear that none of the information I am presenting in this post should be treated as factual.  This is merely my understanding an obscure and undocumented PeopleTools technology.

References to the %metadata package can be seen in some of the delivered code.  From what I can gather, the delivered code uses this package for accessing/manipulating PeopleTools-managed objects.  This package offers a (potentially safer) alternative to accessing/manipulating PeopleTools objects through the metadata tables in the database.

&nbsp;

**Overview of %metadata**

There are three major aspects to understand when using the %metadata package.

  * **Definitions**
  * **Managers**
  * **Keys**

**Definitions** can be thought of as the object types.  So if you want to work with a record object type, then it would be referenced with %metadata as follows:

<pre><span style="color: #0000ff;"><span style="color: #008000;">/* Record Definition */</span>
Local</span> %metadata:RecordDefn:RecordDefn &oRecordDefn;</pre>

In order to obtain a reference to an object definition, you must use the manager for the given object type. **Managers** are responsible for fetching  and creating object definitions.  So to work with a record definition, the record definition manager must first be instantiated as follows:

<pre><span style="color: #008000;"> /* Record Manager */</span>
<span style="color: #0000ff;"> Local</span> %metadata:RecordDefn:RecordDefn_Manager &oRecordDefn_Manager;
 &oRecordDefn_Manager = <span style="color: #0000ff;">create</span> %metadata:RecordDefn:RecordDefn_Manager();</pre>

In order for the manager to supply a reference to an existing object definition, you will have to supply the manager with the key to the definition.  **Keys** are name-value pairs that reference a single definition of a given object type.  So if we want to manage a record named &#8220;PSM_TEST&#8221;, then the key would be created as follows:

<pre><span style="color: #008000;"> /* Example Key Object representing a record */</span>
 <span style="color: #0000ff;">Local</span> %metadata:Key &oRecordKey;
 &oRecordKey = <span style="color: #0000ff;">create</span> %metadata:Key(Key:Class_Record, <span style="color: #ff0000;">"PSM_TEST"</span>);</pre>

The key object instantiation is rather strange (syntax-wise) and I will get into more detail on this later.  But for now, we can feed the generated key object to the record manager&#8217;s GetDefn method, to obtain a non-updateable reference to the PSM_TEST record object definition.

<pre><span style="color: #008000;">/* Get reference to a non-updateable record definition */</span>
&oRecordDefn = &oRecordDefn_Manager.GetDefn(&oRecordKey);</pre>

If you want to make changes to the record definition, then you can obtain an updatable version of the definition by supplying the generated key to the manager&#8217;s GetDefnToUpdate method.

<pre><span style="color: #008000;">/* Get reference to an updatable record definition */</span>
&oRecordDefn = &oRecordDefn_Manager.GetDefnToUpdate(&oRecordKey);</pre>

As I said before, managers are capable of creating new object definitions.  The key is not needed for creating a new object.  The manager&#8217;s CreateDefn method will return a reference to a new (empty) record definition.

<pre><span style="color: #008000;">/* Get reference to a new record definition */</span>
&oRecordDefn = &oRecordDefn_Manager.CreateDefn();</pre>

Now that we have a high level overview of the important pieces to using %metadata, I would like to go into more granular detail of each of the pieces so that we can understand how to actually manipulate PeopleTools-managed objects.

&nbsp;

**Understanding %metadata Definitions**

%metadata has numerous sub-packages that  each represent a PeopleTools-managed object type.  I gave an example of accessing the Record object type above by referencing the RecordDefn sub-package. Many (if not all) object types can be referenced with %metadata and they are all referenced by a somewhat guessable sub-package naming scheme.  Here are some examples:

<pre><span style="color: #008000;"> /* Example Definition Objects */</span>
 <span style="color: #0000ff;">Local</span> %metadata:MenuDefn:MenuDefn &oMenuDefn;
 <span style="color: #0000ff;">Local</span> %metadata:ComponentDefn:ComponentDefn &oComponentDefn;
 <span style="color: #0000ff;">Local</span> %metadata:PageDefn:PageDefn &oPageDefn;
 <span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram &oPeopleCodeProgram;
 <span style="color: #0000ff;">Local</span> %metadata:AppPackageDefn:AppPackageDefn &oAppPackageDefn;
 <span style="color: #0000ff;">Local</span> %metadata:RoleDefn:RoleDefn &oRoleDefn;
 <span style="color: #0000ff;">Local</span> %metadata:PermissionListDefn:PermissionListDefn &oPermissionListDefn;</pre>

The key takeaway to understanding these objects is that they essentially represent their database &#8220;*DEFN&#8221; table counterpart.  For example,  in order to understand the %metadata Record Definition, you must first understand the structure of the PSRECDEFN table in the database.  The properties of the Record Definition object should, more or less, reflect the fields that are on the PSRECDEFN table.

&nbsp;

**Understanding %metadata Managers**

I have found that each %metadata object definition package contains a manager class.  The exact name of the manger class for any given object definition seems to be the object definition name with &#8220;\_Manager&#8221; appended to it.   As we saw earlier the manger class name for the RecordDefn Record object definition was RecordDefn\_Manager.  Here are some examples of other types of managers:

<pre><span style="color: #008000;"> /* Example Manager Objects */</span>
 <span style="color: #0000ff;">Local</span> %metadata:MenuDefn:MenuDefn_Manager &oMenuDefn_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:ComponentDefn:ComponentDefn_Manager &oComponentDefn_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:PageDefn:PageDefn_Manager &oPageDefn_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager &oPeopleCodeProgram_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:AppPackageDefn:AppPackageDefn_Manager &oAppPackageDefn_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:RoleDefn:RoleDefn_Manager &oRoleDefn_Manager;
 <span style="color: #0000ff;">Local</span> %metadata:PermissionListDefn:PermissionListDefn_Manager &oPermissionListDefn_Manager;</pre>

All of the manager classes seem to have a common set of methods for getting and creating their respective object definitions.  I demonstrated earlier the GetDefn, GetDefnToUpdate, and CreateDefn methods for the Record Definition manager class.  I have found that these methods are defined in manager classes of other object definitions as well.  Along with these three methods are a coulple of other commom methods: DefnExists and GetPrivateDefn.  I am unsure of use cases for the GetPrivateDefn method, but the DefnExists method can be use to determine if an object defnition exists for a provided %metadata:key.

&nbsp;

**Understanding %metadata Keys**

The keys are used to reference a specific definition for a given object type.  The key object is a required parameter for all manager class&#8217;s GetDefn, GetDefnToUpdate, and DefnExists mehods. The instantiation of a key object is rather strange as the key constructor behaves in an overloaded fashion and takes a non-PeopleCode object type as a parameter.  This can be proven by trying to extend the Key object&#8217;s constructor in App Designer:

[<img class="alignnone size-full wp-image-992" src="/assets/images/2017/04/Key_Constructor.png" alt="Key_Constructor" width="558" height="421" srcset="/assets/images/2017/04/Key_Constructor.png 558w, /assets/images/2017/04/Key_Constructor-300x226.png 300w, /assets/images/2017/04/Key_Constructor-504x380.png 504w" sizes="(max-width: 558px) 100vw, 558px" />](/assets/images/2017/04/Key_Constructor.png)

As you can see from the picture above, the key&#8217;c constructor takes a RepeatedAny object type, which is not a native PeopleCode object type.  This leads me to believe that we are directly referencing a lower-level language (non-PeopleCode) implementation of these (%metadata) objects.  This could be one of the reasons why we do not have access to view the source of the %metadata package in App Designer.

Aside from its odd syntactical references, the key object is fairly straight-forward to understand. The %metadata:key class contains numerous sub-class, integer constants.  Each of the integer constants represent a specific object type. As we saw earlier, I used the Class_Record sub-class to represent the Record Definition object type.  Here are some examples of how to reference other available key sub-classes:

<pre><span style="color: #008000;"> /* Example Key sub-class, integer constants */</span>
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &iMenuObjType = Key:Class_Menu;
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &iComponentObjType = Key:Class_PanelGroup;
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &iMarketObjType = Key:Class_Market;
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &iPageObjType = Key:Class_Panel;
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &iMethodObjType = Key:Class_Method;</pre>

The integers returned by these sub-classes are the key in the key-value pairs to construct a %metadata:key to give to a manager for an object definition.  The value in the key-value pair is simply a string.   Earlier I inputted the string &#8220;PSM\_TEST&#8221; to refer to a record definition of a record named PSM\_TEST.  To further solidify our understanding of this concept, I would like to provide another example of instantiating a key.  This time, I want to refer to a Component-level PeopleCode Program object. Specifically,  I will reference the PostBuild event to the USERMAINT Component:

<pre><span style="color: #008000;"> /* Example Key Object representing a PeopleCode Program */</span>
 <span style="color: #0000ff;">Local</span> %metadata:Key &oKey = <span style="color: #0000ff;">create</span> %metadata:Key(Key:Class_PanelGroup, <span style="color: #ff0000;">"USERMAINT"</span>, Key:Class_Market, <span style="color: #ff0000;">"GBL"</span>, Key:Class_Method, <span style="color: #ff0000;">"PostBuild"</span>);</pre>

A similar way to generate keys is to use the AddItem method.  This is my preferred way to generate keys as it is a bit easier to read and understand.  Here is an example of using AddItem to generate a key that references the same Component-level PeopleCode Program object as in the previous example:

<pre><span style="color: #008000;"> /* Instantiate the Key object */</span>
 <span style="color: #0000ff;">Local</span> %metadata:Key &oKey = <span style="color: #0000ff;">create</span> %metadata:Key();
 
 <span style="color: #008000;">/* Add the Component key-value pair */</span>
 &oKey.AddItem(Key:Class_PanelGroup, <span style="color: #ff0000;">"USERMAINT"</span>);
 
 <span style="color: #008000;">/* Add the Market key-value pair */</span>
 &oKey.AddItem(Key:Class_Market, <span style="color: #ff0000;">"GBL"</span>);
 
 <span style="color: #008000;">/* Add the Event key-value pair */</span>
 &oKey.AddItem(Key:Class_Method, <span style="color: #ff0000;">"PostBuild"</span>);</pre>

The knowledge of which key-value pairs are needed to reference a particular object type can be derived a couple of different ways.  The first way is to consider the key fields of the corresponding &#8220;*DEFN&#8221; database table of the object type that you are referencing.  The second way is to simply think of what input values App Designer requires for you to view a particular object type.

&nbsp;

**A Complete %metadata Example**

I would now like to provide a complete example of using the %metadata application package. In this example, I will be editing the PeopleCode defined in the PostBuild event of the USERMAINT component.

<pre><span style="color: #0000ff;">import</span> %metadata:Key;
<span style="color: #0000ff;">import</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager;
<span style="color: #0000ff;">import</span> %metadata:PeopleCodeProgram:PeopleCodeProgram;

<span style="color: #008000;">/* Instantiate the Key object */</span>
<span style="color: #0000ff;">Local</span> %metadata:Key &oKey = <span style="color: #0000ff;">create</span> %metadata:Key();

<span style="color: #008000;">/* Add the USERMAINT Component key */</span>
&oKey.AddItem(Key:Class_PanelGroup, <span style="color: #ff0000;">"USERMAINT"</span>);

<span style="color: #008000;">/* Add the GBL Market key */</span>
&oKey.AddItem(Key:Class_Market, <span style="color: #ff0000;">"GBL"</span>);

<span style="color: #008000;">/* Add the PostBuild Event Name key */</span>
&oKey.AddItem(Key:Class_Method, <span style="color: #ff0000;">"PostBuild"</span>);

<span style="color: #008000;">/* Instantiate the PeopleCode Program Manager object */</span>
<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager &oManager = <span style="color: #0000ff;">create</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager();

<span style="color: #008000;">/* Determine if a PeopleCode Program Definition exists for the given key */</span>
<span style="color: #0000ff;">Local boolean</span> &bExists = &oManager.DefnExists(&oKey);

<span style="color: #008000;">/* Throw an exception if the definiton does not exists */</span>
<span style="color: #0000ff;">If Not</span> (&bExists) <span style="color: #0000ff;">Then</span>
<span style="color: #0000ff;"> throw CreateException</span>(0, 0, <span style="color: #ff0000;">"Definition does not exist for the provided key"</span>);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #008000;">/* Get the PeopleCode Program Definition */</span>
<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram &oPeopleCodeProgram = &oManager.GetDefnToUpdate(&oKey);

<span style="color: #008000;">/* Get the PeopleCode that is definied for the loaded PeopleCode Program */</span>
<span style="color: #0000ff;">Local string</span> &sProgram = &oPeopleCodeProgram.GetProgram();

<span style="color: #008000;">/* Append a mesasagebox to the obtained PeopleCode string */</span>
&sProgram = &sProgram | <span style="color: #ff0000;">"messagebox(0,"""",0,0, ""Modifying PeopleCode with PeopleCode!"");"</span>;

<span style="color: #008000;">/* Update the PeopleCode for the Peoplecode Program */</span>
<span style="color: #0000ff;">Local any</span> &test1, &test2, &test3;
<span style="color: #0000ff;">Local boolean</span> &bUpdatedPeopleCode = &oPeopleCodeProgram.UpdateProgram(&sProgram, &test1, &test2, &test3);

<span style="color: #008000;">/* Throw an exception if the PeopleCode did not update */</span>
<span style="color: #0000ff;">If Not</span> (&bUpdatedPeopleCode) <span style="color: #0000ff;">Then</span>
<span style="color: #0000ff;"> throw CreateException</span>(0, 0, <span style="color: #ff0000;">"PeopleCode did not update"</span>);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #008000;">/* Update the PeopleCode Program Definition to save the change to the PeopleCode */</span>
<span style="color: #0000ff;">Local boolean</span> &bDefnUpdated = &oPeopleCodeProgram.UpdateDefn();

<span style="color: #008000;">/* Throw an exception if the definition did not update */</span>
<span style="color: #0000ff;">If Not</span> (&bDefnUpdated) <span style="color: #0000ff;">Then</span>
<span style="color: #0000ff;"> throw CreateException</span>(0, 0, <span style="color: #ff0000;">"Definition did not update"</span>);
<span style="color: #0000ff;">End-If</span>;</pre>

&nbsp;

**Another Complete %metadata Example (<span style="color: #ff0000;">Updated: 9/4/17</span>)**

I received a comment below from Jason about building Role definitions with %metadata.  I think this is a good use case of this package as it can allow a developer to administer PeopleSoft Security in an automated, object-oriented fashion.  Below is an example of how to use %metadata to assign a Permission List to a Role definition.  If the provided Role definition does not already exist, then the code will create a new Role definition and assign the Permission List to it.

<pre><span style="color: #0000ff;">import</span> %metadata:Key;
<span style="color: #0000ff;">import</span> %metadata:RoleDefn:RoleDefn_Manager;
<span style="color: #0000ff;">import</span> %metadata:RoleDefn:RoleDefn;
<span style="color: #0000ff;">import</span> %metadata:RoleDefn:Roleclass;

<span style="color: #0000ff;">Local</span> %metadata:RoleDefn:RoleDefn &oRoleDefn;
<span style="color: #0000ff;">Local</span> %metadata:RoleDefn:Roleclass &oRoleclass;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bSaved;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sRoleName = <span style="color: #ff0000;">"PSM_TEMP1"</span>;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sDescr = <span style="color: #ff0000;">"My Descr"</span>;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &DescrLong = <span style="color: #ff0000;">"My Long Descr"</span>;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sRoleClass = <span style="color: #ff0000;">"PTPT1000"</span>;

<span style="color: #008000;">/* Instantiate the Key object */</span>
<span style="color: #0000ff;">Local</span> %metadata:Key &oKey = <span style="color: #0000ff;">create</span> %metadata:Key();

<span style="color: #008000;">/* Add the Role Name key */</span>
&oKey.AddItem(Key:Class_RoleName, &sRoleName);

<span style="color: #008000;">/* Instantiate the Role Defn Manager object */</span>
<span style="color: #0000ff;">Local</span> %metadata:RoleDefn:RoleDefn_Manager &oManager = <span style="color: #0000ff;">create</span> %metadata:RoleDefn:RoleDefn_Manager();

<span style="color: #008000;">/* Determine if a Role Definition exists for the given key */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bExists = &oManager.DefnExists(&oKey);

<span style="color: #0000ff;">If</span> &bExists <span style="color: #0000ff;">Then</span>
 &oRoleDefn = &oManager.GetDefn(&oKey);
 <span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">integer</span> &i;
 <span style="color: #008000;">/* Check if the Permission List is already assigned to the Role */</span>
  <span style="color: #0000ff;">For</span> &i = 1 <span style="color: #0000ff;">To</span> &oRoleDefn.Count_Roleclass
   <span style="color: #0000ff;">If</span> (&oRoleDefn.Get_Roleclass(&i).Classid = &sRoleClass) <span style="color: #0000ff;">Then</span>
    <span style="color: #0000ff;">MessageBox</span>(0, <span style="color: #ff0000;">""</span>, 0, 0, <span style="color: #ff0000;">"Permission List "</span> | &sRoleClass | <span style="color: #ff0000;">" is already assigned to Role "</span> | &sRoleName);
    <span style="color: #0000ff;">Return</span>
   <span style="color: #0000ff;">End</span>-<span style="color: #0000ff;">If</span>;
 <span style="color: #0000ff;">End</span>-<span style="color: #0000ff;">For</span>;
 
 <span style="color: #0000ff;">MessageBox</span>(0, <span style="color: #ff0000;">""</span>, 0, 0, <span style="color: #ff0000;">"Assigning Permission List "</span> | &sRoleClass | <span style="color: #ff0000;">" to existing Role "</span> | &sRoleName);
 <span style="color: #008000;">/* Get updatable Role definition and assign the Permission List to it */</span>
 &oRoleDefn = &oManager.GetDefnToUpdate(&oKey);
 &oRoleclass = &oRoleDefn.Append_Roleclass(&oRoleDefn.Count_Roleclass);
 &oRoleclass.Classid = &sRoleClass;
 &bSaved = &oRoleDefn.UpdateDefn();
<span style="color: #0000ff;">Else</span>
 <span style="color: #0000ff;">MessageBox</span>(0, <span style="color: #ff0000;">""</span>, 0, 0, <span style="color: #ff0000;">"Creating new Role "</span> | &sRoleName | <span style="color: #ff0000;">" and assigning Permission List "</span> | &sRoleClass);
 <span style="color: #008000;">/* Create new Role definition and assign the Permission List to it */</span>
 &oRoleDefn = &oManager.CreateDefn();
 &oRoleDefn.Rolename = &sRoleName;
 &oRoleDefn.Descr = &sDescr;
 &oRoleDefn.Descrlong = &DescrLong;
 &oRoleclass = &oRoleDefn.Append_Roleclass(&oRoleDefn.Count_Roleclass);
 &oRoleclass.Classid = &sRoleClass;
 &bSaved = &oRoleDefn.SaveNewDefn();
<span style="color: #0000ff;">End</span>-<span style="color: #0000ff;">If</span>;

<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> &bSaved <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, <span style="color: #ff0000;">"Error saving Role definition"</span>);
<span style="color: #0000ff;">End</span>-<span style="color: #0000ff;">If</span>;</pre>

&nbsp;

**Yet Another Complete %metadata Example (<span style="color: #ff0000;">Updated: 2/27/18</span>)**

There was an interesting post on the Oracle Developer Community asking how we can [Pragmatically Edit PeopleCode](https://community.oracle.com/thread/4125293).  The user needed a way to dynamically assign PeopleCode programs to Component PreBuild events to overcome an Event Mapping bug.  Here is an example of how we can add a PreBuild event PeopleCode program to the USERMAINT Component.

<pre><span style="color: #0000ff;">import</span> %metadata:Key;
<span style="color: #0000ff;">import</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager;
<span style="color: #0000ff;">import</span> %metadata:PeopleCodeProgram:PeopleCodeProgram;

<span style="color: #008000;">/* Set the Component and Market name that you want to add a PreBuild Event to */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sComponent = <span style="color: #ff0000;">"USERMAINT"</span>;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sMarket = <span style="color: #ff0000;">"GBL"</span>;
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sEvent = <span style="color: #ff0000;">"PreBuild"</span>;

<span style="color: #008000;">/* Instantiate the Key object */</span>
<span style="color: #0000ff;">Local</span> %metadata:Key &oKey = <span style="color: #0000ff;">create</span> %metadata:Key();

<span style="color: #008000;">/* Add the USERMAINT Component key */</span>
&oKey.AddItem(Key:Class_PanelGroup, &sComponent);

<span style="color: #008000;">/* Add the GBL Market key */</span>
&oKey.AddItem(Key:Class_Market, &sMarket);

<span style="color: #008000;">/* Add the PostBuild Event Name key */</span>
&oKey.AddItem(Key:Class_Method, &sEvent);

<span style="color: #008000;">/* Instantiate the PeopleCode Program Manager object */</span>
<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager &oManager = <span style="color: #0000ff;">create</span> %metadata:PeopleCodeProgram:PeopleCodeProgram_Manager();

<span style="color: #008000;">/* Determine if a PeopleCode Program Definition exists for the given key */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bExists = &oManager.DefnExists(&oKey);

<span style="color: #008000;">/* Return if the PeopleCode Program already exists */</span>
<span style="color: #0000ff;">If</span> (&bExists) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, <span style="color: #ff0000;">"Definition already exists"</span>);
 <span style="color: #0000ff;">Return</span>;
<span style="color: #0000ff;">End-If</span>;

<span style="color: #0000ff;">Local</span> %metadata:PeopleCodeProgram:PeopleCodeProgram &oPeopleCodeProgram = &oManager.CreateDefn();

<span style="color: #008000;">/* Set the properties of the PeopleCode Program */</span>
&oPeopleCodeProgram.LastUpdDttm = <span style="color: #0000ff;">%Datetime</span>;
&oPeopleCodeProgram.LastUpdOprId = <span style="color: #0000ff;">%OperatorId</span>;

&oPeopleCodeProgram.ObjectID#0# = Key:Class_PanelGroup;
&oPeopleCodeProgram.ObjectValue#0# = &sComponent;

&oPeopleCodeProgram.ObjectID#1# = Key:Class_Market;
&oPeopleCodeProgram.ObjectValue#1# = &sMarket;

&oPeopleCodeProgram.ObjectID#2# = Key:Class_Method;
&oPeopleCodeProgram.ObjectValue#2# = &sEvent;

<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">string</span> &sCode = <span style="color: #ff0000;">"/* My Code Goes Here */"</span>;

<span style="color: #008000;">/* Update the PeopleCode for the Peoplecode Program */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">any</span> &test1, &test2, &test3;
<span style="color: #0000ff;">Local boolean</span> &bUpdatedPeopleCode = &oPeopleCodeProgram.UpdateProgram(&sCode, &test1, &test2, &test3);
<span style="color: #008000;">
/* Throw an exception if the PeopleCode did not update */</span>
<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> (&bUpdatedPeopleCode) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, &test1 | <span style="color: #ff0000;">" - Start Pos: "</span> | &test2 | <span style="color: #ff0000;">" End Pos: "</span> | &test3);
<span style="color: #0000ff;">End-If</span>;

<span style="color: #008000;">/* Save the new PeopleCode Program */</span>
<span style="color: #0000ff;">Local</span> <span style="color: #0000ff;">boolean</span> &bDefnSaved = &oPeopleCodeProgram.SaveNewDefn();
<span style="color: #008000;">
/* Throw an exception if the save failed */</span>
<span style="color: #0000ff;">If</span> <span style="color: #0000ff;">Not</span> (&bDefnSaved) <span style="color: #0000ff;">Then</span>
 <span style="color: #0000ff;">throw</span> <span style="color: #0000ff;">CreateException</span>(0, 0, <span style="color: #ff0000;">"Failed to save the new PeopleCode Program definition"</span>);
<span style="color: #0000ff;">End-If</span>;</pre>

* * *

While I did not provide concrete examples of how to manipulate every possible object type with %metadata, I hope that I shined enough light on the subject to provide direction on how to go about manipulating any particular object type.  I believe that after gaining an understanding of the major aspects (Definitions, Managers, and Keys) of %metadata, one can fairly easily stumble their way through the usage of this package.

I think there can be many interesting use cases of the %metadata application package. I am personally putting this package to use by building out a PIA-based (online) PeopleSoft IDE. At the moment my online IDE is just a PeopleCode event editor with a horrible UI, but it is worth mentioning that I have had great success so far with using this package to view/update PeopleCode on any of the PepleCode events.  My biggest hurdle at the moment is coming with a JavaScript-based PeopleCode syntax highlighter/parser.  Here is a post documenting the [Online PeopleCode Editor Project](https://www.peoplesoftmods.com/tips-and-tricks/online-peoplecode-editor-project/).