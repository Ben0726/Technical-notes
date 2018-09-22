@[TOC](AUDIO POWER LIMIT)

# Overview

The audio limiter serves to prevent the signal level from exceeding a preset limit. It is a closed-loop control includes DSP，Power source and Power amplifier and will protect the IPX[power amplifier equipment] from being overdriven.

# Description
## Diagram
![Audio Power Limit Diagram](https://img-blog.csdn.net/20180922100005528?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
**Key Point**：DSP controls the input audio signal to make the amplifier output audio signal big or small and the power source support equipment power. But if the output signal power is too big it will make the power source voltage down at the same time there will be a large current  through the equipment and what's worse it may burned equipment. So it needs the DSP to get the power source status to make the output audio signal power smaller than the limit value. Power source will give three status to DSP and let DSP know how to control the input audio signal.

 - 11: Power source voltage in the safe range and DSP needs release input signal and increase gain until it big as 1；
 - 01: Power source voltage in the safe range but very approximate limit value and DSP needs do nothing but keep the gain value；
 - 00: Power source voltage in dangerous and needs DSP to press input signal and decrease gain.

## Mathematical principle
![iteration method](https://img-blog.csdn.net/20180922113403719?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - At the limit range，different load has dfferent balance point that the output signal power can be as bigger as possible but will not burned the equipment；
 - The output signal power RMS at the balance point will be the ultimate RMS and if output signal RMS is bigger than ultimate RMS the DSP should press the signal to make the signal RMS below the ultimate RMS that will protect the audio equipment from being overdriven；
 - It uses iteration method to calculate signal RMS like the picture shows and it calculates 1024 point sample to get the signal RMS.

## Program 

 - Astep : gain increase rate； Dstep: gain decrease rate；
 - The significant thing is to set Astep and Dstep value，Astep initialize to be 0.0001 and Dstep initialize to be 0.001；
 - As test performance shows Dstep should be dynamic regulation，if the power source voltage attenuation speed fast Dstep value should be bigger.

```c
/**
  * @brief  PowerLimiter function
  * @param  output and input and length channel_num
  * @retval None
  */
void entry_PwrLimExec(float *output, float *input, unsigned int length, int channel_num)
{
    int i;
    getLimSig();
    for (i = 0; i < length; i++)
    {
        output[i] = input[i] * power_gain_val[0];
        output[i + length] = input[i + length] * power_gain_val[1];
    }
    calcRMS(output,length,channel_num);                     /*calcRMS() need to run continuously every time*/
    get_PwrLim_param(channel_num);
}

/**
  * @brief  get Power Limiter param
  * @param  channel_num
  * @retval None
  */
void get_PwrLim_param(int channel_num)
{
    int i;
    getMidRMS(channel_num);
    //cntflag ++;                         /*count time*/
    for(i = 0; i < channel_num; i++)
    {
        /*get press parameter*/
        if(CurRMS[i][0] > (MidRMS[i]))
        {
            power_gain_val[i] -= Dstep;
            if(power_gain_val[i] <= Dstep)
            {
                power_gain_val[i] = Dstep;
            }
        }
        if(CurRMS[i][0] < (MidRMS[i]))
        {
            power_gain_val[i] += Astep;
            if(power_gain_val[i] >= 1)
            {
                power_gain_val[i] = 1;
            }
        }
    }
}

/**
  * @brief  get MidRMS
  * @param  channel_num
  * @retval None
  */
void getMidRMS(int channel_num)
{
    int i;
    for(i = 0; i < channel_num; i++)
    {
        if(pCurStatus == PRESS)
        {
            MidRMS[i] = (Dgain * CurRMS[i][0]);
        }
        if(pCurStatus == RELEASE)
        {
            MidRMS[i] = (Again * CurRMS[i][0]);
        }
        if(pCurStatus == KEEP)
        {
            MidRMS[i] = CurRMS[i][0];
        }
    }
}
/**
  * @brief  calculate RMS value
  * @param input, length, channel_num
  * @retval None
  */
void calcRMS(float *input, unsigned int length, int channel_num)
{
    int i,j;
    static float RMS0square[CHN_MAX];
    static float RMS1square[CHN_MAX];
    for(j = 0; j < channel_num; j++)
    {
        for (i = 0; i < length; i++)                                    /*calculate one block(32 point) RMS*/
        {
            RMS0square[j] = delta * input[j * length + i] * input[j * length + i] + mdelta * PreRMS[j] * PreRMS[j];
            CurRMS[j][i] = sqrt(RMS0square[j]);
            PreRMS[j] = CurRMS[j][i];
        }
    }
}
```

# Test Performance

 - Picture 1 : Sine input signal voltage 1.5V and 4R load，dynamic Dstep maybe from 0.001 to 0.02 and Astep is fixed to be 0.0001；
 - Picture 2：Sine input signal voltage 1.3V and 4R load，Dstep fixed to be 0.002 and Astep is fixed to be 0.0001；
 - Picture 3：Burst input signal and max voltage 1.5V and min voltage 0.5V and 4R load and as the hardware designed it will not trigger the DSP to press the signal.

![1.5V sine input signal 4R load](https://img-blog.csdn.net/20180922152657969?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![1.3V sine input signal 4R load](https://img-blog.csdn.net/20180922152758693?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 ![Burst signal 4R load](https://img-blog.csdn.net/20180922152816353?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbnRlbmdkZW1h/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

