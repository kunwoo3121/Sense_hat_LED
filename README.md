# Sense_hat_LED

* 라즈베리파이와 Sense hat을 이용하여 LED에 원하는 색의 빛을 내게 하는 프로그램을 작성한다.

## 소스 코드
```.c
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
#include<sys/mman.h>
#include<stdint.h>
#include<string.h>
#include<linux/fb.h>
#include<sys/ioctl.h>

#define FILEPATH "/dev/fb0"
#define NUM_WORDS 64
#define FILESIZE (NUM_WORDS * sizeof(uint16_t))

#define RGB565_RED 0xF800
#define RGB565_GREEN 0x07D0
#define RGB565_BLUE 0x001F

void delay(int);

int main(void)
{
	int i;
	int fbfd;
	uint16_t *map;
	uint16_t *p;
	struct fb_fix_screeninfo fix_info;

	fbfd = open(FILEPATH, O_RDWR);

	if(fbfd == -1)
	{
		perror("Error call to open");
		exit(EXIT_FAILURE);
	}

	if(ioctl(fbfd, FBIOGET_FSCREENINFO, &fix_info) == -1)
	{
		perror("Error call to ioctl");
		close(fbfd);
		exit(EXIT_FAILURE);
	}

	if(strcmp(fix_info.id, "RPi-Sense FB") != 0)
	{
		printf("%s\n", "Error: RPi-Sense FB not found");
		close(fbfd);
		exit(EXIT_FAILURE);
	}

	map = mmap(NULL, FILESIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fbfd, 0);

	if(map == MAP_FAILED)
	{
		close(fbfd);
		perror("Error mmapping the file");
		exit(EXIT_FAILURE);
	}

	p = map;

	memset(map, 0, FILESIZE);

	for(i = 0; i < NUM_WORDS; i++)
	{
		*(p + i) = RGB565_RED;
		delay(25);
	}

	for(i = 0; i < 3; i++)
	{
		delay(250);
		memset(map, 0xFF, FILESIZE);
		delay(250);
		memset(map, 0, FILESIZE);
	}
	
	delay(250);
	
	memset(map, 0, FILESIZE);

	if(munmap(map, FILESIZE) == -1)
	{
		perror("Error un-mmapping the file");
	}
	close(fbfd);

	return 0;
}

void delay(int t)
{
	usleep(t * 1000);
}
```

## 동작 내용
```
Sense Hat의 LED는 8*8 총 64개로 구성되어있다

이 64개의 LED를 원하는 색으로 (현재 RED로 지정) 차례차례 밝힌 뒤

흰 색으로 3번 켜졌다가 꺼진다.
```

## 동작 사진
![1](https://user-images.githubusercontent.com/28796089/100021975-baf1ab80-2e25-11eb-85c5-47ab85ff3587.jpg)  
![2](https://user-images.githubusercontent.com/28796089/100021978-bc22d880-2e25-11eb-9977-84cd1021b5cd.jpg)  
```
빨간 색으로 채워지고 흰 색으로 켜지는 모습을 볼 수 있다.
```
![3](https://user-images.githubusercontent.com/28796089/100021980-bc22d880-2e25-11eb-9f15-99b09aca9071.jpg)  
![4](https://user-images.githubusercontent.com/28796089/100021984-bcbb6f00-2e25-11eb-83f5-23ea78b6da60.jpg)  
```
다른 색으로도 동작시켜보았다.

초록색에 해당하는 값을 RGB_GREEN, 파란색에 해당하는 값을 RGB_BLUE로 정의하였다.
```


