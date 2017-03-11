using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Drawing;
using System.Runtime.InteropServices;
using System.Windows.Interop;
using System.ComponentModel;
using System.Threading;
using OpenHardwareMonitor.Hardware;
using System.Reflection;
using System.Windows.Forms;
using IniParser;
using IniParser.Model;

namespace SimpleTempLoadOverlay
{
    /// <summary>
    /// Interaktionslogik für MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window, INotifyPropertyChanged
    {
        //Read Ini
        FileIniDataParser parser = new FileIniDataParser();
        IniData inidata = new IniData();

        //settings Dictionary
        Dictionary<string, System.Windows.Forms.MenuItem> SettingsDictionary = new Dictionary<string, System.Windows.Forms.MenuItem>();


        public System.Windows.Media.Color TextColor = Colors.White;

        //init Context Menu
        System.Windows.Forms.ContextMenu contextMenu1 = new System.Windows.Forms.ContextMenu();

        //init Positions
        System.Windows.Forms.MenuItem PositionMenuItem = new System.Windows.Forms.MenuItem("Position");
        System.Windows.Forms.MenuItem PositionMenuItemTopLeft = new System.Windows.Forms.MenuItem("TopLeft");
        System.Windows.Forms.MenuItem PositionMenuItemTopRight = new System.Windows.Forms.MenuItem("TopRight");
        System.Windows.Forms.MenuItem PositionMenuItemBottomLeft = new System.Windows.Forms.MenuItem("BottomLeft");
        System.Windows.Forms.MenuItem PositionMenuItemBottomRight = new System.Windows.Forms.MenuItem("BottomRight");

        //init Opacity
        System.Windows.Forms.MenuItem OpacityMenuItem = new System.Windows.Forms.MenuItem("Opacity");
        System.Windows.Forms.MenuItem Opacity000MenuItem = new System.Windows.Forms.MenuItem("0%");
        System.Windows.Forms.MenuItem Opacity025MenuItem = new System.Windows.Forms.MenuItem("25%");
        System.Windows.Forms.MenuItem Opacity050MenuItem = new System.Windows.Forms.MenuItem("50%");
        System.Windows.Forms.MenuItem Opacity075MenuItem = new System.Windows.Forms.MenuItem("75%");
        System.Windows.Forms.MenuItem Opacity100MenuItem = new System.Windows.Forms.MenuItem("100%");


        //init Text Color
        System.Windows.Forms.MenuItem TextColorMenuItem = new System.Windows.Forms.MenuItem("Text Color");
        System.Windows.Forms.MenuItem TextColorWhiteMenuItem = new System.Windows.Forms.MenuItem("White");
        System.Windows.Forms.MenuItem TextColorGreenMenuItem = new System.Windows.Forms.MenuItem("Green");
        System.Windows.Forms.MenuItem TextColorRedMenuItem = new System.Windows.Forms.MenuItem("Red");
        System.Windows.Forms.MenuItem TextColorBlueMenuItem = new System.Windows.Forms.MenuItem("Blue");

        //init Exit
        System.Windows.Forms.MenuItem ExitMenuItem = new System.Windows.Forms.MenuItem("Exit");

        public event PropertyChangedEventHandler PropertyChanged;

        private void OnPropertyChanged(string info)
        {
            PropertyChangedEventHandler handler = PropertyChanged;
            if (handler != null)
            {
                handler(this, new PropertyChangedEventArgs(info));
            }
        }

        Canvas Canvas1 = new Canvas();
        Window overlay = new Window();

        TextBlock cpuload;
        TextBlock cputemp;
        TextBlock gputemp;



        Computer cpt = new Computer();
        public static int[,] data;
        public static int numberOfCores = Environment.ProcessorCount;
        public static bool twoGPUs = false;


        public MainWindow()
        {
            InitializeComponent();

        }

        /// <summary>
        public enum SetWindowPosFlags : uint
        {
            SWP_ASYNCWINDOWPOS = 0x4000,
            SWP_DEFERERASE = 0x2000,
            SWP_DRAWFRAME = 0x0020,
            SWP_FRAMECHANGED = 0x0020,
            SWP_HIDEWINDOW = 0x0080,
            SWP_NOACTIVATE = 0x0010,
            SWP_NOCOPYBITS = 0x0100,
            SWP_NOMOVE = 0x0002,
            SWP_NOOWNERZORDER = 0x0200,
            SWP_NOREDRAW = 0x0008,
            SWP_NOREPOSITION = 0x0200,
            SWP_NOSENDCHANGING = 0x0400,
            SWP_NOSIZE = 0x0001,
            SWP_NOZORDER = 0x0004,
            SWP_SHOWWINDOW = 0x0040,
        }
        public static class HWND
        {
            public static IntPtr
            NoTopMost = new IntPtr(-2),
            TopMost = new IntPtr(-1),
            Top = new IntPtr(0),
            Bottom = new IntPtr(1);
        }

        [DllImport("user32.dll")]
        [return: MarshalAs(UnmanagedType.Bool)]
        static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, int X, int Y, int cx, int cy, SetWindowPosFlags uFlags);
        /// </summary>
        public const int WS_EX_TRANSPARENT = 0x00000020;
        public const int GWL_EXSTYLE = (-20);

        [DllImport("user32.dll")]
        public static extern int GetWindowLong(IntPtr hwnd, int index);

        [DllImport("user32.dll")]
        public static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);

        protected override void OnSourceInitialized(EventArgs e)
        {
            base.OnSourceInitialized(e);

            // Get this window's handle
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            IntPtr hwnd2 = new WindowInteropHelper(overlay).Handle;

            // Change the extended window style to include WS_EX_TRANSPARENT
            int extendedStyle = GetWindowLong(hwnd, GWL_EXSTYLE);
            SetWindowLong(hwnd, GWL_EXSTYLE, extendedStyle | WS_EX_TRANSPARENT);

            int extendedStyle2 = GetWindowLong(hwnd2, GWL_EXSTYLE);
            SetWindowLong(hwnd2, GWL_EXSTYLE, extendedStyle2 | WS_EX_TRANSPARENT);
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {



            Border roundBorder1 = new Border();
            roundBorder1.BorderThickness = new Thickness(5);
            roundBorder1.CornerRadius = new CornerRadius(15);



            var notifyIcon = new System.Windows.Forms.NotifyIcon
            {
                Visible = true,
                Icon = System.Drawing.Icon.ExtractAssociatedIcon(Assembly.GetExecutingAssembly().Location),
                Text = Title
            };

            //Startup Message
            notifyIcon.ShowBalloonTip(1, "SimpleTempLoadOverlay", "Hardware Monitoring started", System.Windows.Forms.ToolTipIcon.Info);





            //Add Positions to Position Menu
            PositionMenuItem.MenuItems.AddRange(new System.Windows.Forms.MenuItem[] { PositionMenuItemTopLeft, PositionMenuItemTopRight, PositionMenuItemBottomLeft, PositionMenuItemBottomRight });

            //Add PositionMenu to Mainmenu
            contextMenu1.MenuItems.Add(PositionMenuItem);



            //Add Opacitys to Opacity Menu
            OpacityMenuItem.MenuItems.AddRange(new System.Windows.Forms.MenuItem[] { Opacity000MenuItem, Opacity025MenuItem, Opacity050MenuItem, Opacity075MenuItem, Opacity100MenuItem });

            //Add Opacity to Main Menu
            contextMenu1.MenuItems.Add(OpacityMenuItem);



            //Add Text Colors to Text Color Menu
            TextColorMenuItem.MenuItems.AddRange(new System.Windows.Forms.MenuItem[] { TextColorWhiteMenuItem, TextColorGreenMenuItem, TextColorRedMenuItem, TextColorBlueMenuItem });

            //Add Text Colors to Main Menu
            contextMenu1.MenuItems.Add(TextColorMenuItem);



            //Add Exit to MainMenu
            contextMenu1.MenuItems.Add(ExitMenuItem);

            //Add MainMenu to Icon
            notifyIcon.ContextMenu = contextMenu1;

            //Add Position Handlers
            PositionMenuItemTopRight.Click += new System.EventHandler(PositionMenuItemTopRight_Click);
            PositionMenuItemTopLeft.Click += new System.EventHandler(PositionMenuItemTopLeft_Click);
            PositionMenuItemBottomLeft.Click += new System.EventHandler(PositionMenuItemBottomLeft_Click);
            PositionMenuItemBottomRight.Click += new System.EventHandler(PositionMenuItemBottomRight_Click);

            //Add Opacity Handlers
            Opacity000MenuItem.Click += new System.EventHandler(OpacityMenuItem_Click000);
            Opacity025MenuItem.Click += new System.EventHandler(OpacityMenuItem_Click025);
            Opacity050MenuItem.Click += new System.EventHandler(OpacityMenuItem_Click050);
            Opacity075MenuItem.Click += new System.EventHandler(OpacityMenuItem_Click075);
            Opacity100MenuItem.Click += new System.EventHandler(OpacityMenuItem_Click100);

            //Add Text Color Handlers
            TextColorWhiteMenuItem.Click += new System.EventHandler(TextColorWhiteMenuItem_Click);
            TextColorGreenMenuItem.Click += new System.EventHandler(TextColorGreenMenuItem_Click);
            TextColorRedMenuItem.Click += new System.EventHandler(TextColorRedMenuItem_Click);
            TextColorBlueMenuItem.Click += new System.EventHandler(TextColorBlueMenuItem_Click);

            //Add Exit Handler
            ExitMenuItem.Click += new System.EventHandler(ExitMenuItem_Click);


            //Settings Dictionary
            SettingsDictionary.Add("0.00", Opacity000MenuItem);
            SettingsDictionary.Add("0.25", Opacity025MenuItem);
            SettingsDictionary.Add("0.50", Opacity050MenuItem);
            SettingsDictionary.Add("0.75", Opacity075MenuItem);
            SettingsDictionary.Add("1.00", Opacity100MenuItem);

            SettingsDictionary.Add("Top-Left", PositionMenuItemTopLeft);
            SettingsDictionary.Add("Top-Right", PositionMenuItemTopRight);
            SettingsDictionary.Add("Bottom-Left", PositionMenuItemBottomLeft);
            SettingsDictionary.Add("Bottom-Right", PositionMenuItemBottomRight);

            SettingsDictionary.Add("White", TextColorWhiteMenuItem);
            SettingsDictionary.Add("Red", TextColorRedMenuItem);
            SettingsDictionary.Add("Green", TextColorGreenMenuItem);
            SettingsDictionary.Add("Blue", TextColorBlueMenuItem);



            try
            {
                inidata = parser.ReadFile("Configuration.ini");
            }
            catch (Exception)
            {
                IniData defaultValues = new IniData();
                defaultValues["General"]["Position"] = "Top-Left";
                defaultValues["General"]["Text-Color"] = "white";
                defaultValues["General"]["Opacity"] = "0.75";

                try
                {
                    parser.WriteFile("Configuration.ini", defaultValues);
                }
                catch (Exception)
                {


                }
                inidata = defaultValues;
            }

            string PositionFromSettings = inidata["General"]["Position"];
            string TextColorFromSettings = inidata["General"]["Text-Color"];
            string OpacityFromSettings = inidata["General"]["Opacity"];


            //Set Opacity from Settings


            //set Standard Ticks for Menu
           // System.Windows.Forms.MenuItem defValuePos = PositionMenuItemTopLeft;
            //tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)defValuePos, (System.Windows.Forms.MenuItem)defValuePos.Parent);

            System.Windows.Forms.MenuItem defValueColor = TextColorWhiteMenuItem;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)defValueColor, (System.Windows.Forms.MenuItem)defValueColor.Parent);

            //System.Windows.Forms.MenuItem defValueOpacity = Opacity075MenuItem;
            //tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)defValueOpacity, (System.Windows.Forms.MenuItem)defValueOpacity.Parent);







            overlay.Background = System.Windows.Media.Brushes.Transparent;
            overlay.Top = 0;
            overlay.Width = 85;
            overlay.Height = 50;
            overlay.Left = 0;
            overlay.WindowStyle = WindowStyle.None;
            overlay.Topmost = true;
            overlay.Content = Canvas1;
            //Grid1.Children.Add(Canvas1);
            overlay.AllowsTransparency = true;
            overlay.IsHitTestVisible = false;
            overlay.ShowInTaskbar = false;
            cpuload = Text(1, 1, TextColor);
            cputemp = Text(1, 16, TextColor);
            gputemp = Text(1, 31, TextColor);
            overlay.Show();


            try
            {
                SettingsDictionary[(string)OpacityFromSettings].PerformClick();
                SettingsDictionary[(string)PositionFromSettings].PerformClick();
                SettingsDictionary[(string)TextColorFromSettings].PerformClick();

            }
            catch (Exception)
            {

                
            }

            Thread mainloop = new Thread(Loop);
            mainloop.Name = "UpdateValues";
            mainloop.IsBackground = true;
            mainloop.Priority = ThreadPriority.BelowNormal;
            mainloop.Start();



        }

        private void TextColorBlueMenuItem_Click(object sender, EventArgs e)
        {
            changeTextColor(Colors.Blue);
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Text-Color"] = "Blue";
            parser.WriteFile("Configuration.ini", inidata);

        }



        private void TextColorRedMenuItem_Click(object sender, EventArgs e)
        {
            changeTextColor(Colors.OrangeRed);
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Text-Color"] = "Red";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void TextColorGreenMenuItem_Click(object sender, EventArgs e)
        {
            changeTextColor(Colors.LightGreen);
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Text-Color"] = "Green";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void TextColorWhiteMenuItem_Click(object sender, EventArgs e)
        {
            changeTextColor(Colors.White);
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Text-Color"] = "White";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void tickCurrentItemAndUnceckRest(System.Windows.Forms.MenuItem selectedMenuItem, System.Windows.Forms.MenuItem ParentMenuItem)
        {
            foreach (System.Windows.Forms.MenuItem Item in ParentMenuItem.MenuItems)
            {
                Item.Checked = false;
            }
            selectedMenuItem.Checked = true;
        }

        private void changeTextColor(System.Windows.Media.Color Color1)
        {
            cpuload.Foreground = new SolidColorBrush(Color1);
            cputemp.Foreground = new SolidColorBrush(Color1);
            gputemp.Foreground = new SolidColorBrush(Color1);
        }
        #region Opacity Event Handler
        private void OpacityMenuItem_Click100(object sender, EventArgs e)
        {
            MainBorder.Opacity = 100;
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Opacity"] = "1.00";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void OpacityMenuItem_Click075(object sender, EventArgs e)
        {
            MainBorder.Opacity = 0.75;
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Opacity"] = "0.75";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void OpacityMenuItem_Click050(object sender, EventArgs e)
        {
            MainBorder.Opacity = 0.50;
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Opacity"] = "0.50";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void OpacityMenuItem_Click025(object sender, EventArgs e)
        {
            MainBorder.Opacity = 0.25;
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Opacity"] = "0.25";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void OpacityMenuItem_Click000(object sender, EventArgs e)
        {
            MainBorder.Opacity = 0;
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Opacity"] = "0.00";
            parser.WriteFile("Configuration.ini", inidata);
        }

        #endregion


        #region Change Display Position Event Handler
        private void PositionMenuItemBottomRight_Click(object sender, EventArgs e)
        {
            changeDisplayPositon("BottomRight");
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Position"] = "Bottom-Right";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void PositionMenuItemBottomLeft_Click(object sender, EventArgs e)
        {
            changeDisplayPositon("BottomLeft");
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Position"] = "Bottom-Left";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void PositionMenuItemTopLeft_Click(object sender, EventArgs e)
        {
            changeDisplayPositon("TopLeft");
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Position"] = "Top-Left";
            parser.WriteFile("Configuration.ini", inidata);
        }

        private void PositionMenuItemTopRight_Click(object sender, EventArgs e)
        {
            changeDisplayPositon("TopRight");
            System.Windows.Forms.MenuItem menuItemInternal = (System.Windows.Forms.MenuItem)sender;
            tickCurrentItemAndUnceckRest((System.Windows.Forms.MenuItem)sender, (System.Windows.Forms.MenuItem)menuItemInternal.Parent);
            inidata["General"]["Position"] = "Top-Right";
            parser.WriteFile("Configuration.ini", inidata);
        }
        #endregion
        private void changeDisplayPositon(string Position)
        {
            if (Position == "TopLeft")
            {
                this.Top = 0;
                this.Left = 0;
                MainBorder.CornerRadius = new CornerRadius(0, 0, 10, 0);
                overlay.Top = 0;
                overlay.Left = 0;

                MakeToolOnTop();
            }
            else if (Position == "TopRight")
            {
                this.Top = 0;
                this.Left = SystemParameters.PrimaryScreenWidth - this.Width;
                MainBorder.CornerRadius = new CornerRadius(0, 0, 0, 10);
                overlay.Left = SystemParameters.PrimaryScreenWidth - overlay.Width;
                overlay.Top = 0;

                MakeToolOnTop();
            }
            else if (Position == "BottomLeft")
            {
                this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
                this.Left = 0;
                MainBorder.CornerRadius = new CornerRadius(0, 10, 0, 0);
                overlay.Left = 0;
                overlay.Top = SystemParameters.PrimaryScreenHeight - overlay.Height;

                MakeToolOnTop();
            }
            else if (Position == "BottomRight")
            {
                this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
                this.Left = SystemParameters.PrimaryScreenWidth - this.Width;
                MainBorder.CornerRadius = new CornerRadius(10, 0, 0, 0);
                overlay.Left = SystemParameters.PrimaryScreenWidth - overlay.Width;
                overlay.Top = SystemParameters.PrimaryScreenHeight - overlay.Height;

                MakeToolOnTop();
            }
        }

        private void ExitMenuItem_Click(object sender, EventArgs e)
        {
            System.Windows.Application.Current.Shutdown();
        }



        private void Loop()
        {
            Pop_Data();
            InitComp(cpt);

            while (true)
            {
                //logik
                //new ManualResetEvent(false).WaitOne(1500);
                //this.Dispatcher.Invoke(() => MakeToolOnTop());
                Update_Values(cpt);
                AvgMinMax();
                Display();

                Thread.Sleep(1500);

            }
        }

        private void MakeToolOnTop()
        {

            overlay.Topmost = true;
            this.Topmost = true;
            var wih = new WindowInteropHelper(this);
            var wih2 = new WindowInteropHelper(overlay);
            IntPtr hWnd2 = wih2.Handle;
            SetWindowPos(wih.Handle, HWND.TopMost, 0, 0, 0, 0, SetWindowPosFlags.SWP_NOMOVE | SetWindowPosFlags.SWP_NOSIZE);
            SetWindowPos(hWnd2, HWND.TopMost, 0, 0, 0, 0, SetWindowPosFlags.SWP_NOMOVE | SetWindowPosFlags.SWP_NOSIZE);

        }

        private TextBlock Text(double x, double y, System.Windows.Media.Color color)
        {

            TextBlock textBlock = new TextBlock();
            textBlock.FontSize = 10;
            //textBlock.Name = tb_name;

            //textBlock.SetBinding(TextBlock.TextProperty, binding);
            //textBlock.Text = ((string)reference);



            textBlock.Foreground = new SolidColorBrush(color);
            textBlock.IsHitTestVisible = false;

            Canvas1.IsHitTestVisible = false;

            Canvas.SetLeft(textBlock, x);

            Canvas.SetTop(textBlock, y);


            Canvas1.Children.Add(textBlock);

            return textBlock;
        }

        public void Pop_Data()
        {

            int[,] tmp = new int[3, 4];

            for (int y = 0; y < 3; y++)
                for (int x = 0; x < 4; x++)
                    if (x != 2)
                    {
                        tmp[y, x] = 0;
                    }
                    else
                    {
                        tmp[y, x] = 100;
                    }

            data = tmp;
        }

        public static void InitComp(Computer cmp)
        {
            cmp.Open();
            cmp.CPUEnabled = true;
            cmp.GPUEnabled = true;
        }

        public static void Update_Values(Computer cmp)
        {

            data[0, 1] = 0;
            data[0, 0] = 0;
            int ct = 0;

            foreach (var hardware in cmp.Hardware)
            {

                hardware.Update();

                if (twoGPUs)
                {
                    if (ct == 0)
                        ct++;
                    else
                        ct--;
                }


                foreach (var sensor in hardware.Sensors)
                {

                    //CPU temps & Loads
                    if (sensor.Hardware.HardwareType == HardwareType.CPU)
                    {
                        if (sensor.SensorType == SensorType.Temperature && sensor.Value != null && sensor.Name != "CPU Package")
                        {
                            data[0, 1] += (int)sensor.Value;
                        }
                        if (sensor.SensorType == SensorType.Load && sensor.Name != "CPU Total" && sensor.Value != null)
                        {
                            data[0, 0] += (int)sensor.Value;
                        }
                    }
                    //GPU Temps & Loads
                    else if (sensor.Hardware.HardwareType == HardwareType.GpuNvidia || sensor.Hardware.HardwareType == HardwareType.GpuAti)
                    {

                        if (twoGPUs)
                        {
                            if (ct != 0)
                            {
                                if (sensor.SensorType == SensorType.Temperature && sensor.Value != null)
                                    data[1, 1] = (int)sensor.Value;
                                if (sensor.SensorType == SensorType.Load && sensor.Name == "GPU Core" && sensor.Value != null)
                                    data[1, 0] = (int)sensor.Value;
                            }
                            else
                            {
                                if (sensor.SensorType == SensorType.Temperature && sensor.Value != null)
                                    data[2, 1] = (int)sensor.Value;
                                if (sensor.SensorType == SensorType.Load && sensor.Name == "GPU Core" && sensor.Value != null)
                                    data[2, 0] = (int)sensor.Value;
                            }
                        }
                        else
                        {
                            if (sensor.SensorType == SensorType.Temperature && sensor.Value != null)
                                data[1, 1] = (int)sensor.Value;
                            if (sensor.SensorType == SensorType.Load && sensor.Name == "GPU Core" && sensor.Value != null)
                                data[1, 0] = (int)sensor.Value;
                        }

                    }

                }
            }


        }

        public static void AvgMinMax()
        {

            //CPU Average, divided by 4 for a 4-cores CPU
            data[0, 1] = data[0, 1] / numberOfCores;
            data[0, 0] = data[0, 0] / numberOfCores;

            //MinMax CPU
            if (data[0, 1] < data[0, 2])
                data[0, 2] = data[0, 1];
            if (data[0, 1] > data[0, 3])
                data[0, 3] = data[0, 1];

            //MinMax GPU1
            if (data[1, 1] < data[1, 2])
                data[1, 2] = data[1, 1];
            if (data[1, 1] > data[1, 3])
                data[1, 3] = data[1, 1];

            //MinMax GPU2
            if (twoGPUs)
            {
                if (data[2, 1] < data[2, 2])
                    data[2, 2] = data[2, 1];
                if (data[2, 1] > data[2, 3])
                    data[2, 3] = data[2, 1];
            }
        }

        public void Display()
        {
            overlay.Dispatcher.BeginInvoke((Action)(() =>
            {

                gputemp.Text = " GPU-Temp: " + data[1, 1].ToString() + " °C";
                //curr_gpu_load = data[1, 0].ToString() + " %";
                cputemp.Text = " CPU-Temp: " + data[0, 1].ToString() + " °C";
                cpuload.Text = " CPU-Load: " + data[0, 0].ToString() + " %";
            }));

        }

    }
}
