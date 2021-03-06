.. _admin_importformat:
Import format
=============

Tuleap is able to import a project with it's content from an XML file. This section
describes what is the content of this file and how to proceed to generate an XML
compatible with the import tool.

As of today, the following things are covered by the import tool:

- user groups definition with members
- trackers with contents and history (except artifact links)

For more information on how to run import/export checkout :ref:`Project export and import <admin_tasks_import_export>`.

Core
----

Core information imported as of today:

- user groups and membership (user are referenced by username or ldapId)

.. sourcecode:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <project>
      <ugroups>
        <ugroup name="Developers" description="">
          <members>
            <member format="username">joey_star</member>
          </members>
        </ugroup>
      </ugroups>
      [... services ...]
    </project>

Within [... services ...] each service can add a node with the content to export.

Trackers
--------

Basics:

- ``<trackers>`` node contains a list of ``<tracker>``
- Within a ``<tracker>`` there is first the structure of the tracker and then the
  data themselves within ``<artifacts>`` node.
- The tracker structure is made of metadata (like ``<name>``), fields (``<formElements>``),
  semantics (``<semantics>``), Workflow & field dependencies (``<rules>``, ``<workflow>``),
  reports (``<reports>``) and permissions (``<permissions>``).
- An ``<artifact>`` is made of ``<changeset>``, each ``<changeset>`` corresponds to a modification
  of the artifact. Order matters! the first <changeset> is the artifact creation.
- A ``<changeset>`` is composed of a ``<comment>`` (can be in ``text`` or ``html`` format) and
  a set of ``<field_change>``. Each ``<field_change>`` refers to a field referenced in
  the ``<formElements>`` section of ``<tracker>``.

Example of a simple tracker with a few fields
`````````````````````````````````````````````

The example below is a simple tracker made of following fields

- Id (type: artifact id)
- Submitted by (type: submitted by)
- Title (type: string, associated to "title" semantic)
- Description (type: text)
- 2 structure fields columns (C1 and C2)
- Product (type: multiselectbox)
- Status (type: selectbox, associated to "status" semantic)

Some insights to better understand how this works:

- L73: definition of status semantic uses reference to field F6680, this will
  automatically refer to the field defined L51. And same applies for values
  considered as "Open" (<open_values>, L78) that uses references V7678, V7679
  and V7680 defined L56-59.
- L146: the artifact creation set a value to this status field (<field_change field_name="status">)
  and the value (<value format="id">7678</value>) refers to V7678 L56.

.. sourcecode:: xml
  :linenos:
  :emphasize-lines: 51,56,57,58,59,78,146

    <?xml version="1.0" encoding="UTF-8"?>
    <project>
      <trackers>
        <tracker id="T239" parent_id="0" instantiate_for_new_projects="1">
          <name><![CDATA[Simple Tracker]]></name>
          <item_name>simple</item_name>
          <description><![CDATA[simple tracker example]]></description>
          <color>inca_silver</color>
          <cannedResponses/>
          <formElements>
            <formElement type="aid" ID="F6683" rank="3">
              <name>id</name>
              <label><![CDATA[Id]]></label>
            </formElement>
            <formElement type="subby" ID="F6684" rank="4">
              <name>submitted_by</name>
              <label><![CDATA[Submitted by]]></label>
            </formElement>
            <formElement type="string" ID="F6677" rank="5">
              <name>title</name>
              <label><![CDATA[Title]]></label>
              <properties size="30"/>
            </formElement>
            <formElement type="text" ID="F6678" rank="11892">
              <name>description</name>
              <label><![CDATA[Description]]></label>
              <properties rows="10" cols="50"/>
            </formElement>
            <formElement type="column" ID="F6681" rank="11893">
              <name>c1</name>
              <label><![CDATA[C1]]></label>
              <formElements>
                <formElement type="msb" ID="F6679" rank="0">
                  <name>product</name>
                  <label><![CDATA[Product]]></label>
                  <properties size="7"/>
                  <bind type="static" is_rank_alpha="0">
                    <items>
                      <item ID="V7675" label="UI" is_hidden="0"/>
                      <item ID="V7676" label="Database" is_hidden="0"/>
                      <item ID="V7677" label="API" is_hidden="0"/>
                    </items>
                  </bind>
                </formElement>
              </formElements>
            </formElement>
            <formElement type="column" ID="F6682" rank="11894">
              <name>c2</name>
              <label><![CDATA[C2]]></label>
              <formElements>
                <formElement type="sb" ID="F6680" rank="0">
                  <name>status</name>
                  <label><![CDATA[Status]]></label>
                  <bind type="static" is_rank_alpha="0">
                    <items>
                      <item ID="V7678" label="New" is_hidden="0"/>
                      <item ID="V7679" label="Under analysis" is_hidden="0"/>
                      <item ID="V7680" label="Under verification" is_hidden="0"/>
                      <item ID="V7681" label="Done" is_hidden="0"/>
                    </items>
                  </bind>
                </formElement>
              </formElements>
            </formElement>
          </formElements>
          <semantics>
            <semantic type="title">
              <shortname>title</shortname>
              <label>Title</label>
              <description>Define the title of an artifact</description>
              <field REF="F6677"/>
            </semantic>
            <semantic type="status">
              <shortname>status</shortname>
              <label>Status</label>
              <description>Define the status of an artifact</description>
              <field REF="F6680"/>
              <open_values>
                <open_value REF="V7678"/>
                <open_value REF="V7679"/>
                <open_value REF="V7680"/>
              </open_values>
            </semantic>
            <semantic type="tooltip"/>
            <semantic type="plugin_cardwall_card_fields"/>
          </semantics>
          <rules>
            <date_rules/>
            <list_rules/>
          </rules>
          <reports>
            <report is_default="0">
              <name>Default</name>
              <description>The system default artifact report</description>
              <criterias>
                <criteria rank="0">
                  <field REF="F6680"/>
                </criteria>
              </criterias>
              <renderers>
                <renderer type="table" rank="0" chunksz="15">
                  <name>Results</name>
                  <columns>
                    <field REF="F6683"/>
                    <field REF="F6677"/>
                    <field REF="F6680"/>
                    <field REF="F6679"/>
                  </columns>
                </renderer>
              </renderers>
            </report>
          </reports>
          <workflow/>
          <permissions>
            <permission scope="tracker" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_ACCESS_FULL"/>
            <permission scope="field" REF="F6683" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6684" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6677" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6677" ugroup="UGROUP_REGISTERED" type="PLUGIN_TRACKER_FIELD_SUBMIT"/>
            <permission scope="field" REF="F6677" ugroup="UGROUP_PROJECT_MEMBERS" type="PLUGIN_TRACKER_FIELD_UPDATE"/>
            <permission scope="field" REF="F6678" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6678" ugroup="UGROUP_REGISTERED" type="PLUGIN_TRACKER_FIELD_SUBMIT"/>
            <permission scope="field" REF="F6678" ugroup="UGROUP_PROJECT_MEMBERS" type="PLUGIN_TRACKER_FIELD_UPDATE"/>
            <permission scope="field" REF="F6679" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6679" ugroup="UGROUP_REGISTERED" type="PLUGIN_TRACKER_FIELD_SUBMIT"/>
            <permission scope="field" REF="F6679" ugroup="UGROUP_PROJECT_MEMBERS" type="PLUGIN_TRACKER_FIELD_UPDATE"/>
            <permission scope="field" REF="F6680" ugroup="UGROUP_ANONYMOUS" type="PLUGIN_TRACKER_FIELD_READ"/>
            <permission scope="field" REF="F6680" ugroup="UGROUP_REGISTERED" type="PLUGIN_TRACKER_FIELD_SUBMIT"/>
            <permission scope="field" REF="F6680" ugroup="UGROUP_PROJECT_MEMBERS" type="PLUGIN_TRACKER_FIELD_UPDATE"/>
          </permissions>
          <artifacts>
            <artifact id="445">
              <changeset>
                <submitted_by format="username">vaceletm</submitted_by>
                <submitted_on format="ISO8601">2015-11-10T09:05:19+01:00</submitted_on>
                <comments/>
                <field_change field_name="title" type="string">
                  <value><![CDATA[A demo bug]]></value>
                </field_change>
                <field_change field_name="description" type="text">
                  <value format="text"><![CDATA[With some content]]></value>
                </field_change>
                <field_change field_name="product" type="list" bind="static">
                  <value format="id">7675</value>
                </field_change>
                <field_change field_name="status" type="list" bind="static">
                  <value format="id">7678</value>
                </field_change>
              </changeset>
              <changeset>
                <submitted_by format="username">vaceletm</submitted_by>
                <submitted_on format="ISO8601">2015-11-10T09:05:46+01:00</submitted_on>
                <comments>
                  <comment>
                    <submitted_by format="username">vaceletm</submitted_by>
                    <submitted_on format="ISO8601">2015-11-10T09:05:46+01:00</submitted_on>
                    <body format="text"><![CDATA[Some work done]]></body>
                  </comment>
                </comments>
                <field_change field_name="status" type="list" bind="static">
                  <value format="id">7680</value>
                </field_change>
              </changeset>
            </artifact>
          </artifacts>
        </tracker>
      </trackers>
    </project>
