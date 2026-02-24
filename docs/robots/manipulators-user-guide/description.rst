Robot Description
===================

This guide explains how to set up a ROS2 robot description package using the Universal Robots (UR) description packages as a reference. 
Robot descriptions are essential in ROS2 because they define the structure, geometry, kinematics, dynamics, and 
visualization details of a robot. They serve as the foundation for simulation, motion planning, visualization in RViz, and integration with 
`ros2_control` for hardware drivers. A well-structured description package ensures compatibility with industrial standards, enabling robots to 
be used across different simulators, controllers, and applications without modification.


Setting up the ROS2 Robot Description package
---------------------------------------------
The best practice is to create a new directory for every new workspace.

.. code-block:: console

    mkdir -p ~/<directory_name>/src
    cd ~/<directory_name>/src

Keep the packages in your workspace into the ``src`` directory. The above code creates a ``src`` directory inside ``<directory_name>`` and navigates into it.

.. note:: 
    Ensure you are still inside ``/<directory_name>/src`` before following the next steps.

Use the following command to generate a new package:


.. code-block:: console

        ros2 pkg create --build-type ament_cmake <package_name>

If the `<package_name>` is for example named as ``my_robot_description``, then the command creates the directory structure and essential files:

.. code-block:: text

        my_robot_description/ 
        ├── CMakeLists.txt 
        ├── package.xml 
        ├── include/my_robot_description/ 
        └── src/

- ``ros2 pkg create`` - The Ros2 CLI tool for creating a new package with the necessary files and structures
- ``--build-type ament_cmake``- sets up the package to be built using CMake, managed by ament, which is ROS 2's build toolchain.

     - You use this build type for description packages, C++ nodes, or launch/configuration-based packages.

     - If your package only contains Python code, use ``--build-type ament_python`` instead.


Adding Dependencies
~~~~~~~~~~~~~~~~~~~~

1. Updating the package.xml
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After creating the package, the next step is to add the required dependencies in the ``package.xml`` file.
Dependencies ensure that all necessary tools and libraries are available when building or running the package.

Open the ``package.xml`` file located inside your package directory and add the following lines under the existing tags:

.. code-block:: xml
    
    <buildtool_depend>ament_cmake</buildtool_depend>
    <!-- Runtime dependencies -->
    <exec_depend>xacro</exec_depend>
    <exec_depend>robot_state_publisher</exec_depend>
    <exec_depend>joint_state_publisher_gui</exec_depend>
    <exec_depend>rviz2</exec_depend>

These dependencies serve the following purposes:

``ament_cmake``
 - Specifies that the package uses the ``ament_cmake`` build system, required for building and installing files.

``xacro``
 - Enables the use of .xacro (XML Macros) files, which allow modular and reusable robot description files.
 - Commonly used to simplify URDFs by including parameters, macros, and reusable components.

``robot_state_publisher``
 - Reads the robot's URDF and publishes the corresponding TF (transform) tree, which defines how the links and joints are connected in space.
 - This is essential for visualizing the robot in RViz or running simulations.

``joint_state_publisher_gui``
 - Provides a GUI tool to adjust joint values interactively, allowing visualization of the robot's movement in RViz.
 - Useful when the robot is not yet connected to real hardware.

``rviz2``
 - Required when launching RViz from your package to visualize the robot description directly.


2. Updating the CMakeLists.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To ensure these directories are installed and accessible after building, modify the ``CMakeLists.txt`` file:

.. code-block:: console

    cmake_minimum_required(VERSION 3.5)
    project(<package_name>)

    find_package(ament_cmake REQUIRED)

    # Install resource directories

    install(
    DIRECTORY urdf meshes launch rviz config
    DESTINATION share/${PROJECT_NAME}
    )

    ament_package()

This ensures that when the package is built, all necessary files (URDFs, meshes, configs, etc.) are copied into the install and made  discoverable by launch files or other packages.


Creating the Required Folders and Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the dependencies are set up, the next step is to create the folder structure for the robot description package.
A well-organized structure ensures that URDF, meshes, and launch files are easy to locate, maintain, and use across different tools and simulators.

Navigate to your package directory and create the following folders:

.. code-block:: console

    cd ~/<directory_name>/src/<package_name>
    mkdir -p urdf/inc meshes/visual meshes/collision launch rviz config

This creates the standard directory structure used by most robot description packages:

.. code-block:: text

    <package_name>/
    ├── urdf/ 
    │   ├── inc/
    ├── meshes/
    ├── launch/
    ├── rviz/
    └── config/

- urdf
     Contains the robot's URDF (Unified Robot Description Format) or Xacro files.
     These describe the robot's structure — links, joints, geometry, and kinematics.

- inc
      Stores smaller modular Xacro files such as links.xacro, joints.xacro, or materials.xacro.
      These files are included into the main URDF/Xacro file using <xacro:include>.
      This modular design improves maintainability for complex robot models. 
- meshes
     Stores 3D models of the robot's parts used for visualization and collision detection. 
- launch
     Contains launch files (usually written in Python) to load and visualize the robot model in RViz or simulators.

- rviz
     Stores RViz configuration files (e.g., camera view, TF tree, display settings) for quick visualization setup.

- config
     Holds configuration files used by other tools or nodes.
     
         Commonly used for:
             - ros2_control settings
             - Controller YAML files
             - Joint limits and transmission definitions
             - Other parameters for simulation or hardware integration


Creating  required files inside URDF
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Step 1 Creating the Xacro file

Inside the ``urdf`` folder create the xacro file ``<filename>.xacro`` which will define all the joints and links which contains all the parameters that can be changed as required

Add the following line to the top of the code.

.. code-block:: console

    <?xml version="1.0"?>
    <robot xmlns:xacro="http://ros.org/wiki/xacro" name="<package_name>">

This declares the robot and enables Xacro macros.

- Step 2: Defining colours  (Materials)

These materials will help to visualise the robot parts in RViz. Example of defining a material is as below:

.. code-block:: console

    <material name="blue">
    <color rgba="0 0 1 1" />
    </material>

- Step 3: Add configurable parameters.
  
  We can define parameters which can be later modified as and when required, making the model configurable.
  For example; we can create a parameter ``link_length`` which can be used to modify the length of a link if requied.

  Below is how to define such a parameter:

.. code-block:: console

    <xacro:property name= "link_length" value="4.0" />

- Step 4: Add the Base Link

.. code-block:: console

    <link name="base_link"/>

This is the root of the robot, a fixed reference point.

- step 5: Defining a link

.. code-block:: console
   
  <link name="link1">
    <inertial>
     <origin xyz="0 0 0" rpy="0 0 0"/>
     <mass value="1"/>
     <inertia ixx="100"  ixy="0"  ixz="0" iyy="100" iyz="0" izz="100" />          
    </inertial>
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0" />
      <geometry>
        <box size="${link_length} ${width} ${width}" />
      </geometry>
      <material name="blue" />
    </visual>
  </link>

- Step : Add joints

.. code-block:: console

    <joint name="fixed_link_1" type="fixed">
        <parent link="base_link" />
        <child link="link1" />
        <origin xyz="0 0 ${z_offset}" rpy="0 0 0" />
    </joint>

This attaches link1 to the base with an offset ``z_offset`` which can be defined as a parameter in ``Step 3``.

- Step 7: Close the Robot tag

.. code-block:: console

    </robot>

Creating a Robot Visualization Launch File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
text
This launch file loads your robot’s URDF/Xacro, starts the required publishers, and opens RViz with the robot model and TF tree.

In ROS 2, a typical view_robot launch” does three things:

    - ``robot_state_publisher`` publishes TF from the URDF.

    - ``joint_state_publisher_gui`` lets you move joints interactively (useful before hardware).

    - ``Rviz2`` visualizes the robot.
     
- Step 1: Create the launch file inside the launch folder
  
.. code-block:: console
  
  cd ~/<directory_name>/src/<package_name>
  touch launch/view_robot.launch.py

- Step 2: Add the Required Modules.
 
Inside the launch file start with adding the required modules. These modules define the launch arguments.

.. code-block:: console

    from launch import LaunchDescription
    from launch.actions import DeclareLaunchArgument
    from launch.substitutions import Command, LaunchConfiguration
    from launch_ros.actions import Node
    from ament_index_python.packages import get_package_share_directory
    import os

- Step 3: Define the Launch Description function.

.. code-block:: console

    def generate_launch_description():

This is the function that launch system will call when the file is executed.

- Step 4: Locate and process the URDF file using xacro

Use ``PathJoinSubstitution`` to build the path to the ``<package_name>.xacro`` file inside the urdf folder.

.. code-block:: console

        urdf_path = PathJoinSubstitution([
        FindPackageShare('<package_name>'),
        'urdf',
        '<package_name>.xacro'
    ])

Then use the ``xacro`` command to convert it to a proper URDF string at launch time:

.. code-block:: console

        robot_description = Command([
        'xacro', ' ', urdf_path
    ])

This enables parameter substitution and reusability, and ensures that xacro is executed when the robot launches.

- Step 5: Launch ``robot_state_publisher`` , ``joint_state_publisher_gui`` and ``Rviz2``


.. code-block:: console

     Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
        name='joint_state_publisher_gui',
        output='screen'
    )

    # This node provides sliders to simulate joint positions interactively.

    Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description': robot_description}],
        output='screen'
    )

    # This node takes the joint states and URDF and publishes the full robot state to TF

    Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen'
    
    )

    # Launches RViz to visualise the robot.



        return LaunchDescription([
        ... all nodes/actions ...
    ])
    
    # Finally, all defined nodes and actions are put together as part of the launch descrption.


Build and verify installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the file is updated, return to the root of your workspace and build and source the package:

.. code-block:: console

    colcon build
    source install/setup.bash

Now, verify that all files were installed correctly:

.. code-block:: console

    ls install/<package_name>/share/<package_name>

