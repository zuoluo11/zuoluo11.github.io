---
title: window服务守护外部程序不被关闭
date: 2017-01-21 14:34:11
tags:
---
1、需求描述：保证程序能够一直运行，不会关闭。
2、实现过程：创建windows服务并保持自动运行，该服务定时检查程序进程是否关闭，如果关闭则启动。
3、实现过程中遇到的问题：windows服务直接打开外部程序，该程序只存在于任务管理器中的
进程中，界面无法直接显示。

<!--more-->
服务定时访问代码

```C#

public partial class Service1 : ServiceBase
    {
        private System.Timers.Timer aTimer;//定时器
        public Service1()
        {
            InitializeComponent();
        }

        protected override void OnStart(string[] args)
        {
            // As creating a child process might be a time consuming operation,
            // its better to do that in a separate thread than blocking the main thread.
            //System.Threading.Thread ProcessCreationThread = new System.Threading.Thread(MyThreadFunc);
            //ProcessCreationThread.Start();
            MyThreadFunc();
        }
        // This thread function would launch a child process 
        // in the interactive session of the logged-on user.
        public void MyThreadFunc()
        {
            Log.WriteLogToTxt("打开多屏系统");
            aTimer = new System.Timers.Timer();
            //第一次马上打开
            timer1_Tick(null,null);
            //到达时间的时候执行事件；
            aTimer.Elapsed += new ElapsedEventHandler(timer1_Tick);
            // 设置引发时间的时间间隔 此处设置为1秒（1000毫秒） 
            aTimer.Interval = Convert.ToDouble(AppConfig.GetApp("INTERVAL"));
            //设置是执行一次（false）还是一直执行(true)；
            aTimer.AutoReset = true;
            //是否执行System.Timers.Timer.Elapsed事件；
            aTimer.Enabled = true;
            aTimer.Start();

        }
        /// <summary>
        /// 定时事件
        /// </summary>
        /// <param name="source">源对象</param>
        /// <param name="e">ElapsedEventArgs事件对象</param>
        protected void timer1_Tick(object source, ElapsedEventArgs e)
        {
            bool isProcessResult = IsProcessStarted("IocpServer");//判断是否运行
            if (!isProcessResult)
            {
                //Debugger.Launch();调试debug
                Log.WriteLogToTxt("自动开始运行");
                //此处可以直接将exe运行地址赋值如c:\1.exe
                string applicationLocation = AppConfig.GetApp("ApplicationLocation");
                Log.WriteLogToTxt(applicationLocation);
                CreateProcessAsUserWrapper.LaunchChildProcess(applicationLocation);
            }
        }

        private void myprocess_Exited(object sender, EventArgs e)//被触发的程序
        {
            Log.WriteLogToTxt("关闭多屏系统");
        }

        /// <summary>
        /// 判断程序是否运行
        /// </summary>
        /// <param name="processName"></param>
        /// <returns></returns>
        private bool IsProcessStarted(string processName)
        {
            Process[] temp = Process.GetProcessesByName(processName);

            if (temp.Length > 0)
            {
                string tempName = "";
                foreach (Process process in temp)
                {
                    tempName += process.ProcessName + ";";
                }
                //Log.WriteLogToTxt("程序名称" + tempName);
                return true;
            }
            else
            {
                return false;
            }
        }
        protected override void OnStop()
        {
        }
    }

```
### [微软demo](https://code.msdn.microsoft.com/CSCreateProcessAsUserFromSe-b682134e/sourcecode?fileId=50832&pathId=163624599)
