

# File RobotContainer.cpp

[**File List**](files.md) **>** [**cpp**](dir_fdf2b31f12d3ebb2f617242d0514024b.md) **>** [**RobotContainer.cpp**](RobotContainer_8cpp.md)

[Go to the documentation of this file](RobotContainer_8cpp.md)


```C++
// Copyright (c) FIRST and other WPILib contributors.
// Open Source Software; you can modify and/or share it under the terms of
// the WPILib BSD license file in the root directory of this project.

#include "RobotContainer.h"

#include <frc2/command/Commands.h>
#include <frc2/command/button/RobotModeTriggers.h>

RobotContainer::RobotContainer()
{
    ConfigureBindings();
}

void RobotContainer::ConfigureBindings()
{
    // Note that X is defined as forward according to WPILib convention,
    // and Y is defined as to the left according to WPILib convention.
    drivetrain.SetDefaultCommand(

        // Drivetrain will execute this command periodically
        drivetrain.ApplyRequest([this]() -> auto&& {

            m_driveSpeedMultiplier = speeds::drive::driveSpeedMultiplier; // Drive speed multiplier defined in constants.h
            m_turnSpeedMultiplier = speeds::drive::turnSpeedMultiplier; // Turn speed multiplier defined in constants.h

            if(joystick.RightBumper().Get()){ // Get the state of the right bumper and apply speed changes if bumper is pressed
                m_driveSpeedMultiplier = speeds::drive::turboDriveSpeedMultiplier; // Turbo speed!!!
                m_turnSpeedMultiplier = speeds::drive::turboTurnSpeedMultiplier; // Turbo turn rate!!!
            }

            return drive.WithVelocityX(-joystick.GetLeftY() * MaxSpeed * m_driveSpeedMultiplier) // Drive forward with negative Y (forward)
                .WithVelocityY(-joystick.GetLeftX() * MaxSpeed * m_driveSpeedMultiplier) // Drive left with negative X (left)
                .WithRotationalRate(-joystick.GetRightX() * MaxAngularRate * m_turnSpeedMultiplier); // Drive counterclockwise with negative X (left)
        })
    );

    // Idle while the robot is disabled. This ensures the configured
    // neutral mode is applied to the drive motors while disabled.
    frc2::RobotModeTriggers::Disabled().WhileTrue(
        drivetrain.ApplyRequest([] {
            return swerve::requests::Idle{};
        }).IgnoringDisable(true)
    );

    joystick.A().WhileTrue(drivetrain.ApplyRequest([this]() -> auto&& { return brake; }));
    joystick.B().WhileTrue(drivetrain.ApplyRequest([this]() -> auto&& {
        return point.WithModuleDirection(frc::Rotation2d{-joystick.GetLeftY(), -joystick.GetLeftX()});
    }));

    // Run SysId routines when holding back/start and X/Y.
    // Note that each routine should be run exactly once in a single log.
    (joystick.Back() && joystick.Y()).WhileTrue(drivetrain.SysIdDynamic(frc2::sysid::Direction::kForward));
    (joystick.Back() && joystick.X()).WhileTrue(drivetrain.SysIdDynamic(frc2::sysid::Direction::kReverse));
    (joystick.Start() && joystick.Y()).WhileTrue(drivetrain.SysIdQuasistatic(frc2::sysid::Direction::kForward));
    (joystick.Start() && joystick.X()).WhileTrue(drivetrain.SysIdQuasistatic(frc2::sysid::Direction::kReverse));

    // reset the field-centric heading on left bumper press
    joystick.LeftBumper().OnTrue(drivetrain.RunOnce([this] { drivetrain.SeedFieldCentric(); }));



    drivetrain.RegisterTelemetry([this](auto const &state) { logger.Telemeterize(state); });
}

frc2::CommandPtr RobotContainer::GetAutonomousCommand()
{
    return frc2::cmd::Print("No autonomous command configured");
}
```


