---
layout: post
title:  "Smart Sensors and Actuators"
date:   2018-12-19
categories: jekyll update
author: "sy0227"
---
This is about lecture "Smart Sensors and Actuators".

My team tried to make a guiding robot.

<br> * This is our initial presentation.

<https://www.dropbox.com/s/mcit25dn3jo0dbe/SAA%EC%A4%91%EA%B0%84%EB%B0%9C%ED%91%9C.mp4?dl=0>


<br> * This is our final presentation.

<https://www.dropbox.com/s/7siv05lwap9zkz8/SAA.pdf?dl=0>


<br> * This is a video using encorder for our robot.

<https://www.dropbox.com/s/9ob1dgqglc7w4ye/1.%20%EC%97%94%EC%BD%94%EB%8D%94.mp4?dl=0>


<br> * This is a video using gyro sensor for our robot(not using color sensor).

<https://www.dropbox.com/s/cm5u1izymftraco/2.%EC%9E%90%EC%9D%B4%EB%A1%9C.mp4?dl=0>


<br> * This is a video robot is checking floor.(Not a perfect simulation.)

<https://www.dropbox.com/s/zht7hddunzxip4i/3.%20%EC%B8%B5%EC%88%98%EA%B3%84%EC%82%B0.mp4?dl=0>


<br> * This is our source code.
~~~
#pragma config(Motor, motorB, lm, PIDControl, encoder)
#pragma config(Motor, motorC, rm, PIDControl, encoder)
#pragma config(Sensor, S1, cs1, sensorEV3_Color)
#pragma config(Sensor, S2, cs2, sensorEV3_Color)
#pragma config(Sensor, S3, gs, sensorEV3_Gyro)
#pragma config(Sensor, S4, us, sensorEV3_Ultrasonic)

#define doorDistance 30

int endfloor  = 5;// chage part
int stack[6] = {1000, 2, 800, 1, 1830,0}; // change part
int bound = 50; // chage part
int openState = 0;
int i = 0;

task doorState()
{
	while(1)
	{
		i = getUSDistance(us);
		displayBigTextLine(7,"%d",i);
		if (doorDistance < i)
			openState = 1;
		else
			openState = 0;
	}
}

void moveFloor()
{
	displayBigTextLine(1,"press button"); // erase point
	while(getButtonPress(1) == 0) sleep(10);
	eraseDisplay(); // erase point
	startTask(doorState);
	while(1){
		if(openState == 0)
		{
			clearTimer(T1);
			break;
		}
	}
	int floorNow = 1; // start floor
	int onetime = 1; // totaltime + operation once

	while(1){
		if(openState == 1 && onetime == 1)
			{
				if(179 < time100[T1] && time100[T1] < 188)
					floorNow++;
				else if(200 < time100[T1] && time100[T1] < 210)
					floorNow = floorNow+2;
				else if(230< time100[T1] && time100[T1] <250)
					floorNow = floorNow+3;
				else if(260 < time100[T1] && time100[T1] < 275)
					floorNow = floorNow+4;

				if ( floorNow == endfloor)
				{
					displayBigTextLine(1,"arrive");
					displayBigTextLine(3,"%dF", floorNow);
					playSound(soundBeepBeep);
					break;
				}

				else if( endfloor < floorNow)
				{
					displayBigTextLine(1,"error");
				}
				else
				{
					displayBigTextLine(1,"wait");
					displayBigTextLine(3,"%dF", floorNow);
					while(1){
						if(openState == 0)
							{
								clearTimer(T1);
								onetime = 1;
								break;
							}
					}
				}
			}
	}
	stopTask(doorState);
}



void rightturn()
{
	resetGyro(gs);
	while(getGyroDegrees(gs) < 90)
	{
		setMotorSpeed(lm, 30);
		setMotorSpeed(rm, -30);
	}
}

void leftturn()
{
	resetGyro(gs);
	while(getGyroDegrees(gs) > -90)
	{
		setMotorSpeed(lm, -30);
		setMotorSpeed(rm, 30);
	}
}


void guide()
{
	int curval;
	int i = -1;
	resetGyro(gs);
	while(1)
	{
		i++;
		resetMotorEncoder(lm);
		if(stack[i] == 0)
			break;
		else if(stack[i] == 2)
		{
			rightturn();
			resetGyro(gs);
		}
		else if(stack[i] == 1)
		{
			leftturn();
			resetGyro(gs);
		}
		else
		{
			while(getMotorEncoder(lm) < stack[i])
			{
				curval = getGyroDegrees(gs);
	 			setMotorSpeed(lm, 30 - curval);
				setMotorSpeed(rm, 30 + curval);
			}
		}
	}
	leftturn();
	leftturn();
	resetGyro(gs);
	while(1)
	{
		i--;
		resetMotorEncoder(lm);
		if(i < 0)
			break;
		else if(stack[i] == 2)
		{
			leftturn();
			resetGyro(gs);
		}
		else if(stack[i] == 1)
		{
			rightturn();
			resetGyro(gs);
		}
		else
		{
			while(getMotorEncoder(lm) < stack[i])
			{
				curval = getGyroDegrees(gs);
	 			setMotorSpeed(lm, 30 - curval);
				setMotorSpeed(rm, 30 + curval);
			}
		}
	}
}



void	getElv()
{
	resetMotorEncoder(lm);
	int curval;
	resetGyro(gs);
	while(getMotorEncoder(lm) < 100) // go front
			{
				curval = getGyroDegrees(gs);
	 			setMotorSpeed(lm, 30 - curval);
				setMotorSpeed(rm, 30 + curval);
			}

	rightturn();
	resetGyro(gs);
	resetMotorEncoder(lm);

	while(getMotorEncoder(lm) < 100) // go front
			{
				curval = getGyroDegrees(gs);
	 			setMotorSpeed(lm, 30 - curval);
				setMotorSpeed(rm, 30 + curval);
			}

	rightturn();
	rightturn();
}

void outElv()
{
	int curval;
	resetMotorEncoder(lm);
	resetGyro(gs);

	while(getMotorEncoder(lm) < 100) // go front
			{
				curval = getGyroDegrees(gs);
	 			setMotorSpeed(lm, 30 - curval);
				setMotorSpeed(rm, 30 + curval);
			}

	rightturn();

	int left = getColorReflected(cs1);
	int right = getColorReflected(cs2);

	while(left < bound && right <bound)
	{
		setMotorSpeed(lm, -10);
		setMotorSpeed(rm, -10);
		left = getColorReflected(cs1);
		right = getColorReflected(cs2);
	}

	while(left <= bound || right <=bound)
	{
		if(left <= bound)
		{
			setMotorSpeed(lm, -10);
			setMotorSpeed(rm, 0);
		}
		else
		{
			setMotorSpeed(lm, -10);
			setMotorSpeed(rm, 0);
		}

		left = getColorReflected(cs1);
		right = getColorReflected(cs2);
	}

		setMotorSpeed(lm, 0);
		setMotorSpeed(rm, 0);
		sleep(1000);
}


task main()
{
	getElv();
	moveFloor();
	guide();
	moveFloor();
	outElv();
}

~~~
