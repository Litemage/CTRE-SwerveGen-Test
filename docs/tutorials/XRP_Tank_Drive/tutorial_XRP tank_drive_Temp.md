# XRP Tank Drive Tutorial (C++ Command-Based)

This tutorial will guide you through creating a basic tank drive program for the XRP robot using the C++ **Command-Based** programming framework. A tank drive uses one joystick to control the left side of the drivetrain and another joystick to control the right side.

The Command-Based model organizes code into:
*   **Subsystems:** Represent hardware components (e.g., a `Drivetrain`).
*   **Commands:** Represent actions the robot can take (e.g., `DriveWithJoysticks`).
*   **RobotContainer:** Wires subsystems, commands, and controller inputs together.

> This tutorial assumes you have a basic understanding of the WPILib components covered in the [XRP WPILib Overview](./../XRP_WPILib/tutorial_XRP_WPILib.md).

## 1. The Drivetrain Subsystem

A subsystem is a class that encapsulates all the hardware and logic for a specific part of the robot. First, we'll create a `Drivetrain` subsystem.

### Drivetrain.h
This header file declares the motor objects and the methods the subsystem will provide, such as a method to drive the robot.

```cpp
// filepath: src/main/include/subsystems/Drivetrain.h
#pragma once

#include <frc2/command/SubsystemBase.h>
#include <frc/xrp/XRPMotor.h>

class Drivetrain : public frc2::SubsystemBase {
 public:
  Drivetrain();

  /**
   * Drives the robot using tank-style controls.
   * @param leftSpeed  The speed for the left side of the robot (-1 to 1)
   * @param rightSpeed The speed for the right side of the robot (-1 to 1)
   */
  void TankDrive(double leftSpeed, double rightSpeed);

 private:
  frc::XRPMotor m_left_motor{0};
  frc::XRPMotor m_right_motor{1};
};
```

### Drivetrain.cpp
This file contains the implementation. The constructor is used for one-time setup, like inverting a motor.

```cpp
// filepath: src/main/cpp/subsystems/Drivetrain.cpp
#include "subsystems/Drivetrain.h"

Drivetrain::Drivetrain() {
  // Invert the right motor so that positive values on both motors move the robot forward
  m_right_motor.SetInverted(true);
}

void Drivetrain::TankDrive(double leftSpeed, double rightSpeed) {
  m_left_motor.Set(leftSpeed);
  m_right_motor.Set(rightSpeed);
}
```

## 2. The RobotContainer

The `RobotContainer` class is where all the pieces are tied together. It instantiates the subsystems and the controller and sets up the command bindings.

### RobotContainer.h
Declare your subsystems and controller here.

```cpp
// filepath: src/main/include/RobotContainer.h
#pragma once

#include <frc/XboxController.h>
#include <frc2/command/CommandPtr.h>

#include "subsystems/Drivetrain.h"

class RobotContainer {
 public:
  RobotContainer();
  frc2::CommandPtr GetAutonomousCommand();

 private:
  // Subsystems
  Drivetrain m_drivetrain;

  // Driver Controller
  frc::XboxController m_controller{0};

  void ConfigureBindings();
};
```

### RobotContainer.cpp
In the constructor, we will set the **default command** for the `Drivetrain`. A default command runs automatically whenever no other command requiring that subsystem is running. For driving, this is perfect.

We use a `RunCommand` to create a command inline. This command will continuously call our `TankDrive` method with the latest joystick values.

```cpp
// filepath: src/main/cpp/RobotContainer.cpp
#include "RobotContainer.h"

#include <frc2/command/Commands.h>

RobotContainer::RobotContainer() {
  ConfigureBindings();
}

void RobotContainer::ConfigureBindings() {
  // Set the default command for the drivetrain to be a tank drive.
  // This will run whenever no other command is scheduled that requires the drivetrain.
  m_drivetrain.SetDefaultCommand(frc2::RunCommand(
      [this] {
        // Get the joystick values
        // The Y-axis is inverted, so we negate it to make forward positive
        double left = -m_controller.GetLeftY();
        double right = -m_controller.GetRightY();

        // Call the TankDrive method on our drivetrain subsystem
        m_drivetrain.TankDrive(left, right);
      },
      {&m_drivetrain} // Command requires the drivetrain subsystem
  ));
}

frc2::CommandPtr RobotContainer::GetAutonomousCommand() {
  // For this example, we will do nothing in autonomous.
  return frc2::cmd::None();
}
```

## 3. The Main Robot Class

With the Command-Based framework, the main `Robot` class becomes very simple. Its only job is to create the `RobotContainer` and run the command scheduler. The default template provided by WPILib handles this for you.

### Robot.h
```cpp
// filepath: src/main/include/Robot.h
#pragma once

#include <frc/TimedRobot.h>
#include <frc2/command/Command.h>

#include "RobotContainer.h"

class Robot : public frc::TimedRobot {
 public:
  void RobotPeriodic() override;
  void AutonomousInit() override;
  void AutonomousPeriodic() override;
  void TeleopInit() override;
  void TeleopPeriodic() override;
  void TestPeriodic() override;

 private:
  RobotContainer m_container;
  frc2::Command* m_autonomousCommand = nullptr;
};
```

### Robot.cpp
```cpp
// filepath: src/main/cpp/Robot.cpp
#include "Robot.h"

#include <frc/smartdashboard/SmartDashboard.h>
#include <frc2/command/CommandScheduler.h>

void Robot::RobotPeriodic() {
  frc2::CommandScheduler::GetInstance().Run();
}

void Robot::AutonomousInit() {
  m_autonomousCommand = m_container.GetAutonomousCommand().release();
  if (m_autonomousCommand) {
    m_autonomousCommand->Schedule();
  }
}

void Robot::AutonomousPeriodic() {}

void Robot::TeleopInit() {
  if (m_autonomousCommand) {
    m_autonomousCommand->Cancel();
    m_autonomousCommand = nullptr;
  }
}

void Robot::TeleopPeriodic() {}
void Robot::TestPeriodic() {}

#ifndef RUNNING_FRC_TESTS
int main() {
  return frc::StartRobot<Robot>();
}
#endif
```
