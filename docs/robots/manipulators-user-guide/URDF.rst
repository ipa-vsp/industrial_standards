Xacro
~~~~~~  

1. Introduction to Xacro
------------------------


1.1 What is Xacro?
^^^^^^^^^^^^^^^^^^


A **Xacro** (XML Macros) file is an extended form of URDF that supports macros, variables, and mathematical operations, making robot descriptions modular and easier to maintain.  

1.2 Reference Documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The full explanation of Xacro syntax and advanced constructs, we can refer the ROS tutorial for:
`Using Xacro to clean up your code <https://docs.ros.org/en/jazzy/Tutorials/Intermediate/URDF/Using-Xacro-to-Clean-Up-a-URDF-File.html>`_

2. Recommended Xacro File Structure (Two-File Approach)
-------------------------------------------------------

2.1 Main Entry File and Macro Library File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When building modular robot descriptions in ROS 2, it is a **best practice** to separate our robot model into **two main Xacro files**:

1. **A main entry file** → `<robot_name>.urdf.xacro`  
2. **A macro definition file** → `<robot_name>_macro.xacro`

This separation helps keeps the  model **organized**, **reusable**, and **scalable** for different robot configurations.

2.2 Purpose of Each File
^^^^^^^^^^^^^^^^^^^^^^^^


.. table:: Purpose of Each File
    :align: center

    +----------------------------+--------------------+--------------------------------------------------+---------------------------------------------+
    | File                       | Role               | Contains                                         | Used For                                    |
    +============================+====================+==================================================+=============================================+
    | `<robot_name>_macro.xacro` | Library file       | Defines robot structure in a ``<xacro:macro>``   | Template; not run directly.                 |
    +----------------------------+--------------------+--------------------------------------------------+---------------------------------------------+
    | `<robot_name>.urdf.xacro`  | Main entry point   | Includes the macro and instantiates it with      | Entry file to generate URDF for RViz or     |
    |                            |                    | parameters (YAML files, constants, etc.)         | simulation.                                 |
    +----------------------------+--------------------+--------------------------------------------------+---------------------------------------------+

2.3 Why Use Two Files?
^^^^^^^^^^^^^^^^^^^^^^

Using two files provides several benefits:

-  **Modularity** - The robot structure is defined once (in the macro) and reused for multiple variants. 
-  **Reusability** - The same macro can be used with different parameters (link lengths, joint limits, visuals).  
-  **Clarity** - The main `.urdf.xacro` file remains short and readable; it only handles includes and arguments.  
-  **Parameterization** - We can pass different YAML configuration files for different robots without duplicating the entire URDF.

1. Xacro File Structure and Required Elements
---------------------------------------------

A Xacro file expands into a valid URDF. To avoid parsing errors and to 
keep the robot description clean and scalable, the Xacro files should 
follow a consistent structure. This section explains the required elements and the recommended layout used in most ROS 2 robot description packages.

3.1 XML Declaration
^^^^^^^^^^^^^^^^^^^
The XML declaration is typically placed at the very top of the file:


Example:

.. code-block:: xml

    <?xml version="1.0"?>

**Is it mandatory?**

Not always. Many ROS 2 tools can still process the file without it, 
but it is recommended because it keeps the file standards-compliant and improves compatibility with editors and formatting tools.

3.2 `<robot>` Tag and Xacro Namespace
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Every URDF/Xacro file must have a single root ``<robot>`` tag.  
To use Xacro features such as macros, properties, and includes, the Xacro namespace must also be declared.

Example:

.. code-block:: xml

    <robot name="my_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">
        ...
    </robot>

**Explanation:**

- ``name="my_robot"`` defines the robot name used in the generated URDF.
- ``xmlns:xacro="http://www.ros.org/wiki/xacro"`` enables the use of Xacro tags such as:

  - ``<xacro:include>``
  - ``<xacro:property>``
  - ``<xacro:arg>``
  - ``<xacro:macro>``

If the namespace is missing, Xacro tags will not be recognized and the file will fail during expansion.


3.3 Including External Files (`xacro:include`)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


In ROS 2, robot descriptions are typically split into multiple Xacro files to keep them modular.  
For example, we can separate:

    - robot geometry macros
    - material/color definitions
    - gazebo simulation tags
    - ros2_control transmissions

These files can be imported using ``<xacro:include>``.

Example:

.. code-block:: xml

    <xacro:include filename="$(find-pkg-share my_robot_description)/urdf/my_robot_macro.xacro"/>

We can include multiple files if needed:

.. code-block:: xml

    <xacro:include filename="$(find-pkg-share my_robot_description)/urdf/materials.xacro"/>
    <xacro:include filename="$(find-pkg-share my_robot_description)/urdf/my_robot_macro.xacro"/>

**Best practices for includes:**

    - Place include statements near the top of the file, right after the ``<robot>`` tag.
    - Keep include statements grouped by purpose (e.g., materials, macros, gazebo extensions).
    - Use meaningful filenames to make the structure easy to understand.

3.4 Recommended Ordering Inside a Xacro File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A clean and readable Xacro file follows a consistent order so that all variables and macros are defined before they are used.  
The recommended order for most entry Xacro files is:

1. XML declaration
2. Root ``<robot>`` tag with Xacro namespace
3. Includes (``xacro:include``)
4. Arguments (``xacro:arg``)
5. Properties (``xacro:property``)
6. Macro definitions (``xacro:macro``) *(usually stored in the macro file)*
7. Macro instantiation (creating the final robot model)

A minimal example layout is shown below:

.. code-block:: xml

    <?xml version="1.0"?>
    <robot name="my_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">

        <!-- Includes -->
        <xacro:include filename="$(find-pkg-share my_robot_description)/urdf/my_robot_macro.xacro"/>

        <!-- Arguments -->
        <xacro:arg name="prefix" default=""/>

        <!-- Properties -->
        <xacro:property name="base_length" value="0.30"/>

        <!-- Calling the macro -->
        <xacro:my_robot prefix="$(arg prefix)"/>

    </robot>

**Why this ordering is recommended:**

   - Includes must appear before any macros defined inside them are called.
   - Arguments and properties should be defined early so they are available throughout the file.
   - The use macro is placed near the end to keep the entry file short and readable.

Following this structure makes it easier to expand the robot description 
later (for example, adding meshes, colors, ros2_control, or gazebo simulation tags).

4. Xacro Inputs and Macro Parameters (Link Function Inputs)
-----------------------------------------------------------

Xacro allows robot descriptions to be flexible and reusable by supporting **external inputs**, **internal variables**, and **macros with parameters**.  
This helps reduce repeated URDF code and makes it easier to update link dimensions, masses, and naming consistently across the model.

In general, Xacro supports three common ways to manage values:

   - ``xacro:arg`` → external inputs (set from launch/command line)
   - ``xacro:property`` → internal variables/constants (defined inside the file)
   - Macro parameters → inputs passed into a macro when we call it


4.1 Using `xacro:arg` (External Inputs)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``xacro:arg`` is used for values that we want to change **from outside the Xacro file**, for example:

   - adding a prefix for multi-robot setups
   - enabling/disabling simulation-specific blocks
   - selecting different robot configurations

Example:

.. code-block:: xml

    <xacro:arg name="prefix" default=""/>

The argument can then be used inside the file as:

.. code-block:: xml

    <link name="$(arg prefix)base_link"/>

**When to use `xacro:arg`:**

   - when the value should be configurable without editing the file
   - when the same robot description is reused in multiple setups

4.2 Using `xacro:property` (Internal Variables)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``xacro:property`` is used to define values **inside the Xacro file**, such as link dimensions, masses, offsets, or constants.

Example:

.. code-block:: xml

    <xacro:property name="link_length" value="0.30"/>
    <xacro:property name="link_radius" value="0.02"/>
    <xacro:property name="link_mass" value="0.50"/>

Properties can be used using the ``${...}``  

expression syntax:

.. code-block:: xml

    <cylinder length="${link_length}" radius="${link_radius}"/>

**When to use `xacro:property`:**

    - when values are part of the robot design.
    - to avoid repeating numbers.
    - to compute derived values once and reuse them.
  
4.3 Defining a Macro (`xacro:macro`)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


A macro is a reusable block of URDF that works like a template.  
We define a macro once and call it multiple times to generate similar links and joints.

Basic macro definition example:

.. code-block:: xml

    <xacro:macro name="sample_link" params="link_name">
      <link name="${link_name}"/>
    </xacro:macro>

**Why macros are useful:**

    - avoids repeated code.
    - improves readability.
    - makes robot models scalable and easier to modify.

4.4 Macro Parameters (`params=...`)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Macro parameters define what inputs a macro expects.  
These parameters are listed in the ``params=`` attribute and can then be used inside the macro body.

Example:

.. code-block:: xml

    <xacro:macro name="cylinder_link" params="link_name length radius mass">
      <link name="${link_name}">
        <visual>
          <geometry>
            <cylinder length="${length}" radius="${radius}"/>
          </geometry>
        </visual>
      </link>
    </xacro:macro>

**Best practices for parameter naming:**

    - use descriptive names like ``link_name``, ``length``, ``radius``, ``mass``.
    - keep naming consistent across macros.
    - avoid unclear names unless the macro is very small.

4.5 Calling a Macro (Passing Inputs)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


After defining a macro, we must **call it** to generate URDF output.

Example:

.. code-block:: xml

    <xacro:cylinder_link
      link_name="link_1"
      length="0.30"
      radius="0.02"
      mass="0.50"/>

This call will expand into a valid URDF ``<link>`` element when the Xacro file is processed.

**Important note:**  
If macros are defined but never called, the generated URDF will not contain those links or joints.

We can also pass values using properties:

.. code-block:: xml

    <xacro:cylinder_link
      link_name="link_1"
      length="${link_length}"
      radius="${link_radius}"
      mass="${link_mass}"/>

4.6 Example: Link Macro With Inputs (Name, Length, Radius, Mass)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following example shows a link macro that takes common "link inputs" and generates a cylindrical link with visual, collision, and inertial blocks.

**Macro definition:**

.. code-block:: xml

    <xacro:macro name="cylinder_link" params="link_name length radius mass">
      <link name="${link_name}">

        <visual>
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <geometry>
            <cylinder length="${length}" radius="${radius}"/>
          </geometry>
          <material name="grey">
            <color rgba="0.6 0.6 0.6 1.0"/>
          </material>
        </visual>

        <collision>
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <geometry>
            <cylinder length="${length}" radius="${radius}"/>
          </geometry>
        </collision>

        <!-- Inertial values shown here are simplified for demonstration -->
        <inertial>
          <mass value="${mass}"/>
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <inertia ixx="0.001" ixy="0.0" ixz="0.0"
                   iyy="0.001" iyz="0.0"
                   izz="0.001"/>
        </inertial>

      </link>
    </xacro:macro>

**Calling the macro with inputs:**

.. code-block:: xml

    <xacro:cylinder_link link_name="link_1" length="0.30" radius="0.02" mass="0.50"/>
    <xacro:cylinder_link link_name="link_2" length="0.25" radius="0.02" mass="0.45"/>

This approach is useful when building modular robots where multiple links share the same structure but have different dimensions or masses.

5. Best Practices for Naming Links and Joints
---------------------------------------------

Consistent naming of links and joints is essential for building robot models that are easy to debug, 
easy to scale, and compatible with common ROS 2 tools such as TF, RViz, MoveIt, and ros2_control.  

A good naming convention makes it clear what each part represents and avoids confusion when working with controllers, transforms, and planning configurations.

5.1 Link Naming Conventions
^^^^^^^^^^^^^^^^^^^^^^^^^^^

A **link** represents a rigid body in the robot model. Link names should be:

   - descriptive and consistent
   - lowercase with underscores.
   - stable 
  
**Recommended link naming patterns:**

   - ``base_link``  
     Main reference link for the robot model (most common root link).
   - ``shoulder_link``, ``upper_arm_link``, ``forearm_link``, ``wrist_link``. Clear and readable naming for arm-type robots.

   - ``tool0``  
     Common industrial robot tool frame naming (often used in UR robots).

**Best practice:**  
Use the suffix ``_link`` consistently for all rigid bodies.

5.2 Joint Naming Conventions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A **joint** defines the relationship between two links and specifies how motion occurs (fixed, revolute, continuous, prismatic).  
Joint names should be consistent, descriptive, and match the physical robot structure.

**Recommended joint naming patterns:**

   - ``shoulder_joint``, ``elbow_joint``, ``wrist_joint``  
     Clear naming for human-readable arm structure.
   - ``joint_1``, ``joint_2``, ``joint_3``  
     Acceptable for simple serial chains.
   - ``base_to_shoulder_joint``  
     Useful when explicitly describing the connection between parent and child.

**Best practice:**  
Use the suffix ``_joint`` for all joints and keep naming consistent across the full robot chain.


5.3 Naming for TF Tree Readability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


In ROS 2, the TF tree is used to represent coordinate transforms between robot frames.  
Link and joint names directly affect TF readability because most frames correspond to link names.

**Recommendations for a clean TF tree:**

   - Keep the root link name clear (typically ``base_link``).
   - Use meaningful names for important frames such as:
     - ``base_link`` (robot base reference)
     - ``ee_link`` or ``tool0`` (end-effector reference)
     - sensor frames like ``camera_link`` or ``lidar_link`` (if applicable)
   - Avoid confusing names like ``link`` or ``joint`` without numbers or context.
   - Keep naming consistent across URDF, TF, and configuration files.

5.4 Common Naming Mistakes to Avoid
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following naming issues commonly lead to confusion or integration problems:

**1. Inconsistent naming style**
Mixing formats such as ``Link1``, ``link_1``, and ``link-1`` reduces readability and causes errors in configs.

**2. Renaming links/joints after configuration**
Changing names later can break:

   - ros2_control controller configuration
   - MoveIt planning groups
   - TF-based tools and scripts

**3. Duplicate names**
Link names and joint names must be unique. Duplicate names can cause URDF parsing failures or TF conflicts.

**4. Using spaces or special characters**
Avoid spaces and characters such as ``-`` or ``/``.  
Stick to lowercase letters, numbers, and underscores.


Following these naming best practices improves clarity, avoids TF confusion, and ensures better compatibility with ROS 2.

6. Adding Color and Materials in Xacro
--------------------------------------

URDF supports visual appearance through the ``<visual>`` element, where we can define geometry and apply colors using ``<material>``.  
In Xacro, materials and colors can be written directly in the URDF blocks, or defined once and reused across multiple links for cleaner and more consistent robot descriptions.


6.1 Visual vs Collision 
^^^^^^^^^^^^^^^^^^^^^^^^

In a URDF/Xacro robot model, each link can contain both:

   - ``<visual>`` → what we see in RViz (appearance)
   - ``<collision>`` → what is used for collision checking (physics and planning)

**Important:**  
Colors and materials only affect the **visual** element. The ``<collision>`` element does not use colors.

Example structure:

.. code-block:: xml

    <link name="example_link">

      <visual>
        <geometry>
          <box size="0.1 0.1 0.1"/>
        </geometry>
      </visual>

      <collision>
        <geometry>
          <box size="0.1 0.1 0.1"/>
        </geometry>
      </collision>

    </link>


6.2 Defining Materials
^^^^^^^^^^^^^^^^^^^^^^

A material in URDF is defined inside the ``<visual>`` tag using the ``<material>`` element.  
Materials can be written in two ways:

**1) Define a material with a color:**

.. code-block:: xml

    <material name="grey">
      <color rgba="0.6 0.6 0.6 1.0"/>
    </material>

**2) Reference an existing material by name:**

.. code-block:: xml

    <material name="grey"/>

A complete example inside a visual block:

.. code-block:: xml

    <visual>
      <geometry>
        <cylinder length="0.2" radius="0.03"/>
      </geometry>
      <material name="grey">
        <color rgba="0.6 0.6 0.6 1.0"/>
      </material>
    </visual>

6.3 Using RGBA Colors
^^^^^^^^^^^^^^^^^^^^^

URDF uses the **RGBA** format to define colors, where each value is **normalized between 0.0 and 1.0**:

   - **0.0** = 0% intensity (none)
   - **1.0** = 100% intensity (maximum)

The RGBA channels mean:

   - **R** = Red (0.0 to 1.0)
   - **G** = Green (0.0 to 1.0)
   - **B** = Blue (0.0 to 1.0)
   - **A** = Alpha / transparency (0.0 to 1.0)

**Alpha (A) values:**

   - ``A = 1.0`` → fully visible (opaque)
   - ``A = 0.0`` → fully invisible (transparent)

**Example RGBA values:**

    - solid red: ``1 0 0 1``
    - solid green: ``0 1 0 1``
    - solid blue: ``0 0 1 1``
    - solid grey: ``0.6 0.6 0.6 1``
    - semi-transparent grey: ``0.5 0.5 0.5 0.5``

**Quick RGBA Color Table:**

+------------+----------------------+
| Color Name | RGBA Value (r g b a) |
+============+======================+
| Black      | ``0 0 0 1``          |
+------------+----------------------+
| White      | ``1 1 1 1``          |
+------------+----------------------+
| Grey       | ``0.6 0.6 0.6 1``    |
+------------+----------------------+
| Red        | ``1 0 0 1``          |
+------------+----------------------+
| Green      | ``0 1 0 1``          |
+------------+----------------------+
| Blue       | ``0 0 1 1``          |
+------------+----------------------+
| Yellow     | ``1 1 0 1``          |
+------------+----------------------+
| Cyan       | ``0 1 1 1``          |
+------------+----------------------+
| Magenta    | ``1 0 1 1``          |
+------------+----------------------+
| Orange     | ``1 0.5 0 1``        |
+------------+----------------------+

Example usage:

.. code-block:: xml

    <material name="blue">
      <color rgba="0 0 1 1"/>
    </material>


6.4 Reusing Materials Across Multiple Links
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To keep the robot description clean, we can define commonly used materials once and reuse them in multiple links.

A common approach is to define the materials at the top of the Xacro file:

.. code-block:: xml

    <material name="grey">
      <color rgba="0.6 0.6 0.6 1.0"/>
    </material>

    <material name="black">
      <color rgba="0.1 0.1 0.1 1.0"/>
    </material>

Then reuse them in visuals:

.. code-block:: xml

    <link name="link_1">
      <visual>
        <geometry>
          <box size="0.1 0.1 0.1"/>
        </geometry>
        <material name="grey"/>
      </visual>
    </link>

    <link name="link_2">
      <visual>
        <geometry>
          <cylinder length="0.2" radius="0.03"/>
        </geometry>
        <material name="black"/>
      </visual>
    </link>

7. Mathematics in Xacro
-----------------------


One of the key advantages of Xacro over plain URDF is that it supports basic mathematical expressions.  
This allows us to compute dimensions, offsets, and positions instead of hardcoding values repeatedly.  
Using math improves readability and reduces errors when we modify robot geometry later.


7.1 Expressions Using `${...}`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


In Xacro, expressions are written inside the ``${...}`` syntax.  
These expressions can be used anywhere a numeric value is expected in URDF (for example in geometry sizes, joint origins, or inertial values).

Example:

.. code-block:: xml

    <xacro:property name="L" value="0.30"/>
    <xacro:property name="R" value="0.02"/>

    <visual>
      <geometry>
        <cylinder length="${L}" radius="${R}"/>
      </geometry>
    </visual>

Xacro supports common arithmetic operators:

   - addition: ``+``
   - subtraction: ``-``
   - multiplication: ``*``
   - division: ``/``

Example:

.. code-block:: xml

    <xacro:property name="half_L" value="${L/2}"/>
    <xacro:property name="double_R" value="${2*R}"/>


7.2 Using Math for Joint Origins and Offsets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Joint origins define where a child link is placed relative to its parent link.  
Instead of manually calculating offsets, we can compute them using Xacro math.

Example: placing a child link at the end of a parent cylinder link:

.. code-block:: xml

    <xacro:property name="parent_length" value="0.30"/>

    <joint name="joint_1" type="revolute">
      <parent link="base_link"/>
      <child link="link_1"/>
      <origin xyz="${parent_length/2} 0 0" rpy="0 0 0"/>
      <axis xyz="0 0 1"/>
    </joint>

In this example:

    - the parent link is assumed to be centered at its own origin
    - the child link is placed at ``parent_length/2`` along the x-axis

This is a common pattern when modeling robot arms where each link origin is located at its center.


7.3 Reusing Computed Values with `xacro:property`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To keep the Xacro file clean, computed values should be stored in ``xacro:property`` variables and reused.  
This avoids repeating expressions in multiple locations and makes updates easier.

Example:

.. code-block:: xml

    <xacro:property name="link_length" value="0.40"/>
    <xacro:property name="link_offset" value="${link_length/2}"/>

    <joint name="joint_2" type="fixed">
      <parent link="link_1"/>
      <child link="link_2"/>
      <origin xyz="${link_offset} 0 0" rpy="0 0 0"/>
    </joint>

**Best practice:**

   - compute once using ``xacro:property``
   - reuse the value in joints, visuals, and collision blocks

7.4 Common Math Examples (Half-length, Symmetry, Scaling)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Math in Xacro is commonly used for geometry alignment and symmetric robot structures.

**1) Half-length offsets (centered geometry)**  
Many URDF geometries are defined about their center. To place the next joint at the end of a link, half-length is often required:

.. code-block:: xml

    <xacro:property name="L1" value="0.30"/>
    <xacro:property name="L1_end" value="${L1/2}"/>

**2) Symmetry (left/right components)**  
For symmetric robots, we can use a sign variable to mirror positions:

.. code-block:: xml

    <xacro:property name="y_offset" value="0.10"/>

    <!-- Left side -->
    <joint name="left_joint" type="fixed">
      <parent link="base_link"/>
      <child link="left_link"/>
      <origin xyz="0 ${y_offset} 0" rpy="0 0 0"/>
    </joint>

    <!-- Right side -->
    <joint name="right_joint" type="fixed">
      <parent link="base_link"/>
      <child link="right_link"/>
      <origin xyz="0 ${-y_offset} 0" rpy="0 0 0"/>
    </joint>

**3) Scaling dimensions**  
Scaling is useful when creating variants of the same robot design:

.. code-block:: xml

    <xacro:property name="scale" value="1.2"/>
    <xacro:property name="base_length" value="${0.30*scale}"/>
    <xacro:property name="base_radius" value="${0.02*scale}"/>

    <link name="scaled_link">
      <visual>
        <geometry>
          <cylinder length="${base_length}" radius="${base_radius}"/>
        </geometry>
      </visual>
    </link>

Using math and properties together makes the robot model easier to modify and reduces the risk of incorrect manual calculations.

8. Adding Mesh Files and Scaling
-----------------------------------

Meshes are commonly used in robot descriptions to represent realistic geometry such as robot arms, grippers, brackets, and sensor housings.  
In URDF/Xacro, meshes are usually added inside the ``<visual>`` tag for display in RViz, and optionally inside the ``<collision>`` tag for collision checking.

8.1 Supported Mesh Formats (STL vs DAE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The most common mesh formats used in ROS robot descriptions are:

- **STL (.stl)**  
   - widely used for 3D printing and CAD export  
   - supports geometry only (no color / no texture)

- **DAE (.dae) / Collada**  
   - supports geometry + materials + textures  
   - preferred when you want textured models in RViz

**Recommendation:**

- Use **STL** for simple models where color is not required.
- Use **DAE** when you need color, textures, or material appearance.
  
8.2 Adding a Mesh to a Link Visual
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Meshes are typically added inside the ``<visual>`` block of a link.

Example (mesh visual):

.. code-block:: xml

    <link name="link_1">
      <visual>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
          <mesh filename="package://my_robot_description/meshes/link_1.stl"/>
        </geometry>
      </visual>
    </link>

**Key points:**

- The ``<origin>`` inside ``<visual>`` defines where the mesh is placed relative to the link frame.
- If the mesh appears rotated or shifted in RViz, you usually need to adjust the ``xyz`` or ``rpy`` values.


8.3 Adding a Mesh to Collision Geometry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Collision geometry is used for physics simulation and collision checking (MoveIt / planning).  

Example (mesh collision):

.. code-block:: xml

    <link name="link_1">
      <collision>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
          <mesh filename="package://my_robot_description/meshes/link_1.stl"/>
        </geometry>
      </collision>
    </link>

8.4 Scaling Meshes in URDF/Xacro
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes meshes appear too large or too small in RViz because of unit mismatches (mm vs m).  
URDF supports scaling meshes using the ``scale`` attribute.

Example:

.. code-block:: xml

    <mesh filename="package://my_robot_description/meshes/link_1.stl"
          scale="0.001 0.001 0.001"/>

**Common scaling values:**

- CAD exported in **millimeters (mm)** → scale by ``0.001 0.001 0.001`` to convert to meters
- CAD exported in **meters (m)** → scale by ``1 1 1`` (no scaling)

**Tip:**  
Always confirm the mesh units in the CAD/Blender export settings to avoid relying on URDF scaling.


8.5 Common Mesh Path Patterns (`package://` and `find-pkg-share`)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In ROS 2 robot descriptions, mesh paths are typically written using  ``package://`` URIs (most common in URDF)


Example:

.. code-block:: xml

    <mesh filename="package://my_robot_description/meshes/link_1.stl"/>

**Recommended folder structure:**

.. code-block:: text

    my_robot_description/
      urdf/
      meshes/
      launch/
      rviz/

**Best practice:**

- Keep all meshes inside the ``meshes/`` folder of your description package.
- Use consistent naming between mesh files and link names.
  
1. Blender Workflow: Adding Color and Texture to Meshes
-------------------------------------------------------

Meshes exported directly from CAD tools often lack color or texture information when used in ROS 2.  
Blender is commonly used as an intermediate tool to apply **materials, colors, and textures** before exporting meshes in a format that RViz can render correctly.

This section explains why some mesh formats do not support color, how to apply materials in Blender, and how to export meshes correctly for use in URDF/Xacro.


9.1 Why STL Does Not Store Color or Texture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The STL file format stores **geometry only**. It does not support:

   - color
   - material properties
   - texture maps

As a result:
   - STL meshes always appear grey in RViz unless a color is applied in URDF using ``<material>``.
   - Any color applied in Blender will be lost if the mesh is exported as STL.

**Implication:**
   - Use STL only for geometry.
   - Use URDF materials for simple coloring.
   - Use a different mesh format if textures are required.


9.2 Recommended Export Formats for Color (DAE / GLB)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To preserve color and texture information, use formats that support materials:

 **DAE (Collada)**  
  - widely supported in ROS and RViz  
  - supports materials and texture images  
  - most commonly used format for textured robot meshes


9.3 Applying Materials and Textures in Blender
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Typical Blender workflow:

1. Import the mesh (STL or other CAD export)
2. Switch to **Material Properties**
3. Create a new material
4. Assign:
   - base color (for simple coloring), or
   - image texture (for detailed appearance)

For textured models:
- Use UV unwrapping
- Assign an image texture to the material’s base color

**Important notes:**

- Each mesh object should have at least one material assigned.
- Avoid procedural textures; use image-based textures instead.
- Keep texture image paths relative (do not use absolute system paths).


9.4 Exporting from Blender for ROS 2 (Scale and Axis Settings)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Correct export settings are critical to avoid scaling and orientation issues in RViz.

**Recommended Blender export settings for DAE:**

- Apply transformations: **Apply All (Ctrl + A → All Transforms)**
- Units:
  - Scene units set to **Metric**
  - Unit scale = **1.0**
- Axis settings:
  - Forward: **-Z Forward**
  - Up: **Y Up**
- Enable:
  - “Apply Modifiers”
  - “Include UV Textures”
  - “Include Materials”

**Why this matters:**

- ROS uses meters as the unit system.
- Incorrect axis mapping leads to rotated or flipped meshes.
- Unapplied transforms cause unexpected scaling in RViz.


9.5 Using Textured Meshes in URDF/Xacro
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Once exported as DAE, the mesh can be used directly in the URDF/Xacro ``<visual>`` block.

Example:

.. code-block:: xml

    <link name="link_1">
      <visual>
        <geometry>
          <mesh filename="package://my_robot_description/meshes/link_1.dae"/>
        </geometry>
      </visual>
    </link>

**Notes:**

- When using textured meshes, do **not** override the color using a URDF ``<material>`` block.
- RViz will use the material and texture embedded in the DAE file.

9.6 Common Blender-to-RViz Issues and Fixes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**1) Mesh appears too large or too small**
  - Cause: unit mismatch (mm vs m)
  - Fix: apply scale in Blender and re-export, or use URDF mesh scaling

**2) Mesh appears rotated**
  - Cause: incorrect axis mapping during export
  - Fix: adjust export axis settings or rotate the mesh in Blender and apply transforms

**3) Texture not visible in RViz**
  Possible causes:
    - texture file not exported or not referenced correctly
    - absolute file paths used in Blender
    - unsupported texture format

  Fix checklist:
    - ensure texture images are in the same folder as the DAE file
    - re-export with “Include Materials” enabled
    - use PNG or JPG textures

**4) Mesh appears black or very dark**
  - Cause: missing or incorrect normals
  - Fix: recalculate normals in Blender before export

Following this workflow ensures that colored and textured meshes display correctly in RViz and integrate cleanly with your Xacro-based robot description.


10. Adding Gazebo Simulation Extensions (Plugins, Sensors, Transmissions)
-----------------------------------------------------------------------------


URDF and Xacro describe the robot’s kinematic structure, visuals, and inertial properties.  
However, simulation in Gazebo requires additional information such as:

    - plugins that control the robot behavior
    - sensors that publish simulated data
    - transmissions that connect joints to actuators

These elements are added using Gazebo-specific tags inside the robot description.

10.1 URDF vs Gazebo Tags (What Changes in Simulation)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A standard URDF is sufficient to display a robot in RViz.  
Gazebo simulation, however, requires extra tags that define physical behavior and interfaces.

Examples of simulation-specific data:

    - friction and damping parameters
    - actuator interfaces
    - sensors (camera, lidar, IMU)
    - simulation plugins

Gazebo-specific information is added using the ``<gazebo>`` tag.

Example:

.. code-block:: xml

    <gazebo reference="link_1">
      <material>Gazebo/Blue</material>
    </gazebo>

The ``reference`` attribute specifies which link or joint the Gazebo settings apply to.


10.2 Adding the Gazebo Plugin Block (`<gazebo>`)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Plugins allow Gazebo to simulate robot functionality such as joint control, sensor output, or special behaviors.

A plugin is typically added inside a ``<gazebo>`` block.

Example:

.. code-block:: xml

    <gazebo>
      <plugin name="gazebo_ros_control" filename="libgazebo_ros2_control.so"/>
    </gazebo>

This plugin connects the simulated robot to ROS 2 control interfaces.

**Key points:**

  - Plugins define simulation behavior.
  - They are loaded by Gazebo when the robot is spawned.
  - Many ROS packages provide ready-made Gazebo plugins.

10.3 Adding Sensors in Gazebo (Camera, IMU, Lidar)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sensors are also added inside ``<gazebo>`` blocks associated with a specific link.

Example: camera sensor

.. code-block:: xml

    <gazebo reference="camera_link">
      <sensor name="camera" type="camera">
        <update_rate>30</update_rate>
        <camera>
          <horizontal_fov>1.047</horizontal_fov>
          <image>
            <width>640</width>
            <height>480</height>
          </image>
        </camera>
        <plugin name="camera_controller" filename="libgazebo_ros_camera.so"/>
      </sensor>
    </gazebo>

Example: IMU sensor

.. code-block:: xml

    <gazebo reference="imu_link">
      <sensor name="imu" type="imu">
        <update_rate>100</update_rate>
        <plugin name="imu_plugin" filename="libgazebo_ros_imu_sensor.so"/>
      </sensor>
    </gazebo>

**Best practice:**

    - Create dedicated sensor links (e.g., ``camera_link``, ``imu_link``).
    - Attach sensors to these links rather than embedding them in structural links.

10.4 Adding Joint Transmissions for Simulation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Transmissions define how a joint connects to an actuator.  
They are required for control interfaces and often used by Gazebo and ``ros2_control``.

Example transmission block:

.. code-block:: xml

    <transmission name="joint_1_transmission">
      <type>transmission_interface/SimpleTransmission</type>
      <joint name="joint_1">
        <hardwareInterface>hardware_interface/PositionJointInterface</hardwareInterface>
      </joint>
      <actuator name="joint_1_motor">
        <mechanicalReduction>1</mechanicalReduction>
      </actuator>
    </transmission>

**Why transmissions are important:**

    - connect joints to simulated actuators
    - required for many controllers
    - help define how commands affect joint motion

10.5 Recommended Separation: Gazebo Xacro File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To keep the robot description modular, it is recommended to place Gazebo-specific blocks in a separate Xacro file, for example:

``robot_gazebo.xacro``

This file can include:

    - sensor definitions
    - Gazebo plugins
    - friction/damping parameters

Then include it in the main entry file:

.. code-block:: xml

    <xacro:include filename="$(find-pkg-share my_robot_description)/urdf/robot_gazebo.xacro"/>

This keeps the core robot model independent of simulation-specific details.


10.6 Common Gazebo Simulation Issues and Fixes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


**1) Robot spawns but joints do not move**
    - cause: missing transmissions or control plugin
    - fix: add transmissions and ``gazebo_ros2_control`` plugin

**2) Sensors do not publish data**
    - cause: missing plugin inside sensor block
    - fix: add correct ROS Gazebo sensor plugin

**3) Robot slides or behaves unrealistically**
    - cause: missing friction or damping values
    - fix: add friction parameters inside ``<gazebo>`` blocks

**4) Simulation crashes when spawning robot**
    - cause: invalid plugin path or syntax error
    - fix: verify plugin filename and URDF validity

Separating Gazebo extensions from the core URDF helps maintain a clean robot model while still supporting full simulation capabilities.

11. Adding ROS 2 Control in Xacro
-------------------------------------

ROS 2 Control provides a standardized framework for commanding robot joints and reading their states.  
It enables controllers such as position, velocity, or effort controllers to interface with either:

    - simulated hardware (Gazebo / Ignition)
    - real robot hardware drivers

To use ROS 2 Control, the robot description must include a ``<ros2_control>`` block that defines hardware interfaces and the joints they control.


11.1 What is `ros2_control` and Why It Is Needed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``ros2_control`` separates the robot model from the control implementation by defining:

    - how joints are actuated
    - which command interfaces they accept
    - which state interfaces they publish

Without this block:

  - controllers cannot command the robot
  - joint states cannot be updated correctly
  - Gazebo control plugins may fail to initialize

In short, the URDF describes *what the robot is*, and ``ros2_control`` describes *how it moves*.


11.2 Adding the `<ros2_control>` Block
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``<ros2_control>`` block is added inside the robot description, usually near the end of the file or inside a dedicated Xacro file.

Example:

.. code-block:: xml

    <ros2_control name="MyRobotSystem" type="system">
      <hardware>
        <plugin>gazebo_ros2_control/GazeboSystem</plugin>
      </hardware>
    </ros2_control>

This tells ROS 2 Control which hardware plugin to use.  
For simulation, the Gazebo system plugin is commonly used.

11.3 Defining Hardware Interfaces (Position/Velocity/Effort)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inside the ``<ros2_control>`` block, each controllable joint must be defined along with the interfaces it supports.

Example:

.. code-block:: xml

    <ros2_control name="MyRobotSystem" type="system">

      <hardware>
        <plugin>gazebo_ros2_control/GazeboSystem</plugin>
      </hardware>

      <joint name="joint_1">
        <command_interface name="position"/>
        <state_interface name="position"/>
        <state_interface name="velocity"/>
      </joint>

    </ros2_control>

**Common command interfaces:**

- ``position`` → most common for manipulators
- ``velocity`` → mobile robots or continuous joints
- ``effort`` → torque control

**Common state interfaces:**

- ``position``
- ``velocity``
- ``effort``

11.4 Defining Joint Command and State Interfaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each joint listed in ``ros2_control`` must:

  - exist in the URDF
  - have matching joint names
  - be controllable (not fixed)

Example with multiple joints:

.. code-block:: xml

    <ros2_control name="MyRobotSystem" type="system">

      <hardware>
        <plugin>gazebo_ros2_control/GazeboSystem</plugin>
      </hardware>

      <joint name="joint_1">
        <command_interface name="position"/>
        <state_interface name="position"/>
      </joint>

      <joint name="joint_2">
        <command_interface name="position"/>
        <state_interface name="position"/>
      </joint>

    </ros2_control>

**Important:**  

Joint names must exactly match the names used in the URDF joint definitions.

11.5 Controller Compatibility Requirements (Naming + Limits)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For controllers to work properly, joints must also have:

  - valid joint limits in the URDF
  - consistent naming between URDF, ros2_control, and controller configuration
  - supported joint types (revolute, continuous, prismatic)

Example joint with limits:

.. code-block:: xml

    <joint name="joint_1" type="revolute">
      <parent link="link_0"/>
      <child link="link_1"/>
      <limit lower="-1.57" upper="1.57" effort="10" velocity="2.0"/>
    </joint>

Without limits, many controllers will refuse to start.


