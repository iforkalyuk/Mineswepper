using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.CodeDom;
using System.Windows;
using System.IO;
using System.Runtime.InteropServices;



namespace Minesweeper
{
    class Program
    {

        struct YX
        {
            public int y;
            public int x;
            YX(int y, int x)
            {
                this.y = y;
                this.x = x;
            }
        }
        struct GamerData
        {
            public string name;
            public int totalSeconds;
        }
        class Cell
        {
            public int countMines;
            public bool mine;
            public bool flag;
            public bool isOpen;
            public YX yx;
            public Cell(int y, int x)
            {
                countMines = 0;
                mine = false;
                flag = false;
                isOpen = false;
                yx.y = y;
                yx.x = x;
            }
        }
        class Field
        {
            public int heigh;
            public int width;
            public Cell[,] cells;
            public Field()
            {
                heigh = 11;
                width = 33;
                cells = new Cell[heigh, width];
                for (int i = 1; i < heigh; i++)
                {
                    for (int j = 1; j < width; j++)
                    {
                        cells[i, j] = new Cell(i, j);
                    }
                }
            }
            public Field(int heigh, int width)
            {
                this.heigh = heigh;
                this.width = width;
                cells = new Cell[heigh, width];
                for (int i = 0; i < heigh; i++)
                {
                    for (int j = 0; j < width; j++)
                    {
                        cells[i, j] = new Cell(i, j);
                    }
                }
            }
            public bool isEmpty(int y, int x)
            {
                if (cells[y, x].flag == false && cells[y, x].mine == false/*&& cells[y,x].countMines==0*/)
                {
                    return true;
                }
                return false;
            }
            public void printField()
            {
                Console.SetCursorPosition(0, 0);
                Console.Write((char)9556);
                for (int i = 0; i <= heigh; i++)
                {
                    for (int j = 1; j <= width + 1; j++)
                    {
                        if (j == 1 && i > 0 && i < heigh || j == width + 1 && i > 0 && i < heigh)
                        {
                            Console.Write((char)9553);
                        }

                        else if (i == 0 && j > 0 && j < width || i == heigh && j > 1 && j < width + 1)
                        {
                            Console.Write((char)9552);
                        }
                        else if (i == heigh && j == 1)
                        {
                            Console.Write((char)9562);
                        }
                        else if (i == 0 && j == width)
                        {
                            Console.Write((char)9559);
                        }
                        else if (i == heigh && j == width + 1)
                        {
                            Console.Write((char)9565);
                        }
                        else
                            Console.Write(" ");
                    }
                    Console.WriteLine();
                }
                Console.SetCursorPosition(width + 1, 0);
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.Write("Time:");
                Console.SetCursorPosition(width + 1, 1);
                Console.Write("Count of Mines:");
                Console.SetCursorPosition(width + 1, 3);
                Console.Write("Count of Flags:");
                Console.SetCursorPosition(6, heigh + 1);
                Console.ForegroundColor = ConsoleColor.DarkCyan;
                Console.Write("Move:");
                Console.SetCursorPosition(17, heigh + 1);
                Console.Write("Action:");
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.SetCursorPosition(17, heigh + 2);
                Console.Write("Open cell: Enter");
                Console.SetCursorPosition(17, heigh + 3);
                Console.Write("Set  flag: Space");
                Console.SetCursorPosition(17, heigh + 4);
                Console.Write("Open  all: +");
                Console.SetCursorPosition(0, heigh + 2);
                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.Write("       up\n       ^\n       |\nleft <- ->right\n       |\n       v \n      down");
                Console.SetCursorPosition(0, 0);
            }
        }
        class Game
        {
            Field field;
            public List<GamerData> results;
            int countMinesInField;
            int countFlagsInField;
            int countOpenCells;
            public int time;
            public int minutes;
            public int hours;
            bool gameOver;
            bool win;
            Game()
            {
                field = new Field();
                results = new List<GamerData>();
                countMinesInField = 30;
                gameOver = false;
                win = false;
            }
            void SetMinesInField()
            {
                int y = 1;
                int x = 1;
                for (int i = 0; i < countMinesInField; i++)
                {
                    Random randY = new Random();
                    y = randY.Next(1, 10);
                    Random randX = new Random();
                    x = randX.Next(1, 30);
                    if (field.cells[y, x].mine == false && field.cells[y, x].isOpen == false)
                    {
                        field.cells[y, x].mine = true;
                    }
                    else
                    {
                        i--;
                    }
                }

            }
            void ShowCountOfMinesNearCell()
            {
                for (int i = 1; i < field.heigh; i++)
                {
                    for (int j = 1; j < field.width; j++)
                    {
                        if (field.cells[i, j].mine == false)
                        {
                            if (i > 1 && j > 1 && field.cells[i - 1, j - 1].mine == true)
                                field.cells[i, j].countMines++;

                            if (i > 1 && field.cells[i - 1, j].mine == true)
                                field.cells[i, j].countMines++;

                            if (i > 1 && j < field.width - 1 && field.cells[i - 1, j + 1].mine == true)
                                field.cells[i, j].countMines++;

                            if (j > 1 && field.cells[i, j - 1].mine == true)
                                field.cells[i, j].countMines++;

                            if (j < field.width - 1 && field.cells[i, j + 1].mine == true)
                                field.cells[i, j].countMines++;

                            if (i < field.heigh - 1 && j > 1 && field.cells[i + 1, j - 1].mine == true)
                                field.cells[i, j].countMines++;

                            if (i < field.heigh - 1 && field.cells[i + 1, j].mine == true)
                                field.cells[i, j].countMines++;

                            if (j < field.width - 1 && i < field.heigh - 1 && field.cells[i + 1, j + 1].mine == true)
                                field.cells[i, j].countMines++;
                            // Console.Write($" {field.cells[i, j].countMines}");
                        }
                    }
                }
            }
            void OpenCell(YX yx)
            {
                if (field.cells[yx.y, yx.x].isOpen == true || field.cells[yx.y, yx.x].flag == true)
                    return;
                if (yx.x == 0)
                {
                    field.cells[yx.y, yx.x].isOpen = true;
                    Console.SetCursorPosition(yx.x, yx.y);
                    if (countOpenCells > 0)
                        PrintCell(yx.y, yx.x);
                    if (field.cells[yx.y, yx.x].mine == true)
                    {
                        YX temp;

                        for (temp.y = 0; temp.y < field.heigh; temp.y++)
                        {
                            for (temp.x = 0; temp.x < field.width; temp.x++)
                            {
                                if (field.cells[temp.y, temp.x].mine == true)
                                {
                                    OpenCell(temp);
                                }
                            }
                        }
                        Console.SetCursorPosition(field.width + 1, 5);
                        Console.ForegroundColor = ConsoleColor.Magenta;
                        Console.WriteLine("GAME OVER");
                        Console.ResetColor();
                        Console.SetCursorPosition(0, 20);
                        gameOver = true;
                    }
                    countOpenCells++;
                }
                else
                {
                    field.cells[yx.y, yx.x].isOpen = true;
                    countOpenCells++;
                    if (countOpenCells > 0)
                        PrintCell(yx.y, yx.x);

                    if (field.cells[yx.y, yx.x].mine == true)
                    {
                        YX temp;

                        for (temp.y = 1; temp.y < field.heigh; temp.y++)
                        {
                            for (temp.x = 1; temp.x < field.width; temp.x++)
                            {
                                if (field.cells[temp.y, temp.x].mine == true)
                                {
                                    OpenCell(temp);
                                }
                            }
                        }
                        Console.SetCursorPosition(field.width + 1, 5);
                        Console.ForegroundColor = ConsoleColor.Magenta;
                        Console.WriteLine("GAME OVER");
                        Console.ResetColor();
                        Console.SetCursorPosition(0, 20);
                        gameOver = true;
                    }
                    countOpenCells++;
                }
            }
            void SetFlag(YX yx)
            {
                if (countFlagsInField == countMinesInField && field.cells[yx.y, yx.x].flag == false)
                {
                    return;
                }
                if (field.cells[yx.y, yx.x].flag == true)
                {
                    field.cells[yx.y, yx.x].flag = false;
                    countFlagsInField--;
                    Console.ForegroundColor = ConsoleColor.Cyan;
                    PrintCell(yx.y, yx.x);

                }
                else
                {
                    field.cells[yx.y, yx.x].flag = true;
                    countFlagsInField++;
                    if (countFlagsInField == countMinesInField)
                    {
                        OpenAll();
                        PrintCell(yx.y, yx.x);
                        if (gameOver == false)
                        {
                            gameOver = true;
                            win = true;
                            Console.ForegroundColor = ConsoleColor.Magenta;
                            Console.SetCursorPosition(field.width + 1, 5);
                            Console.WriteLine("Win!!! Win!!!");
                            Console.ResetColor();
                            Console.SetCursorPosition(field.width + 1, 6);
                            Console.Write("Press any key...");
                            Console.ReadKey();
                            WriteRecords();
                            Console.Write("Press any key...");
                            Console.ReadKey();
                        }
                    }
                }
                Console.SetCursorPosition(field.width + 1, 4);
                Console.ForegroundColor = ConsoleColor.Blue;
                Console.Write(countFlagsInField);
                Console.ResetColor();
                Console.SetCursorPosition(yx.x, yx.y);
            }
            bool CheckFlag(YX yx)
            {
                if (field.cells[yx.y, yx.x].flag == field.cells[yx.y, yx.x].mine)
                {
                    return true;
                }
                return false;
            }
            void OpenNext(YX yx)
            {
                int i = yx.y;
                int j = yx.x;
                List<Cell> forCheck = new List<Cell>();
                forCheck.Add(field.cells[yx.y, yx.x]);
                while (forCheck.Count > 0)
                {
                    i = forCheck[0].yx.y;
                    j = forCheck[0].yx.x;
                    if (j < field.width - 1 && field.isEmpty(i, j + 1) && !field.cells[i, j + 1].isOpen && field.cells[i, j].isOpen 
                        && (field.cells[i, j].countMines < 1 && field.cells[i, j + 1].flag == false))
                    {
                        field.cells[i, j + 1].isOpen = true;
                        forCheck.Add(field.cells[i, j + 1]);
                        PrintCell(i, j + 1);
                    }

                    if (i < field.heigh - 1 && field.isEmpty(i + 1, j) && !field.cells[i + 1, j].isOpen && field.cells[i, j].isOpen 
                        && (field.cells[i, j].countMines < 1 && field.cells[i + 1, j].flag == false))
                    {
                        field.cells[i + 1, j].isOpen = true;
                        PrintCell(i + 1, j);
                        forCheck.Add(field.cells[i + 1, j]);                       
                    }

                    if (i > 1 && field.isEmpty(i - 1, j) && !field.cells[i - 1, j].isOpen && field.cells[i, j].isOpen
                        && (field.cells[i, j].countMines < 1&& field.cells[i - 1, j].flag == false))
                    {
                        field.cells[i - 1, j].isOpen = true;
                        PrintCell(i - 1, j);
                        forCheck.Add(field.cells[i - 1, j]);
                    }

                    if (j < field.width - 1 && i > 1 && field.isEmpty(i - 1, j + 1) && !field.cells[i - 1, j + 1].isOpen
                        && field.cells[i, j].isOpen && (field.cells[i, j].countMines < 1 && field.cells[i - 1, j + 1].flag == false))
                    {
                        field.cells[i - 1, j + 1].isOpen = true;
                        forCheck.Add(field.cells[i - 1, j + 1]);
                        PrintCell(i - 1, j + 1);
                    }

                    if (j < field.width - 1 && i < field.heigh - 2 && field.isEmpty(i + 1, j + 1) && !field.cells[i + 1, j + 1].isOpen
                        && field.cells[i, j].isOpen && (field.cells[i, j].countMines < 1 && field.cells[i + 1, j + 1].flag == false))
                    {
                        field.cells[i + 1, j + 1].isOpen = true;
                        forCheck.Add(field.cells[i + 1, j + 1]);
                        PrintCell(i + 1, j + 1);
                    }

                    if (j > 1 && i < field.heigh - 1 && field.isEmpty(i + 1, j - 1) && !field.cells[i + 1, j - 1].isOpen && field.cells[i, j].isOpen
                        && (field.cells[i, j].countMines < 1 && field.cells[i + 1, j - 1].flag == false))
                    {
                        field.cells[i + 1, j - 1].isOpen = true;
                        forCheck.Add(field.cells[i + 1, j - 1]);
                        PrintCell(i + 1, j - 1);
                    }

                    if (j > 1 && i > 1 && field.isEmpty(i - 1, j - 1) && !field.cells[i - 1, j - 1].isOpen && field.cells[i, j].isOpen
                        && (field.cells[i, j].countMines < 1 && field.cells[i - 1, j - 1].flag == false))
                    {
                        field.cells[i - 1, j - 1].isOpen = true;
                        PrintCell(i - 1, j - 1);
                        forCheck.Add(field.cells[i - 1, j - 1]);
                    }

                    if (j > 1 && field.isEmpty(i, j - 1) && !field.cells[i, j - 1].isOpen && field.cells[i, j].isOpen
                        && (field.cells[i, j].countMines < 1 && field.cells[i, j - 1].flag == false))
                    {
                        field.cells[i, j - 1].isOpen = true;
                        forCheck.Add(field.cells[i, j - 1]);
                        PrintCell(i, j - 1);                       
                    }

                    forCheck.RemoveRange(0, 1);
                }
            }
            bool DoneStartStep(YX yx)
            {

                YX temp;
                temp.x = 0;
                temp.y = 0;
                OpenCell(yx);
                System.Threading.TimerCallback tm = new TimerCallback(Count);
                Timer timer = new Timer(tm, temp, 0, 1000);                
                SetMinesInField();
                ShowCountOfMinesNearCell();
                PrintCell(yx.y, yx.x);
                OpenNext(yx);               
                return true;
            }
            YX MoveCursor(int y, int x)
            {
                YX temp;
                temp.x = x;
                temp.y = y;
                while (true)
                {
                    Console.SetCursorPosition(temp.x, temp.y);
                    switch (Console.ReadKey(intercept: true).Key)
                    {
                        case ConsoleKey.RightArrow:
                            if (temp.x < field.width - 1)
                                temp.x = temp.x + 1;
                            break;
                        case ConsoleKey.LeftArrow:
                            if (temp.x > 1)
                                temp.x = temp.x - 1;
                            break;
                        case ConsoleKey.UpArrow:
                            if (temp.y > 1)
                                temp.y = temp.y - 1;
                            break;
                        case ConsoleKey.DownArrow:
                            if (temp.y < field.heigh - 1)
                                temp.y = temp.y + 1;
                            break;
                        case ConsoleKey.Enter:
                            if (countOpenCells == 0)
                            {
                                DoneStartStep(temp);
                                return temp;
                            }
                            OpenCell(temp);
                            if (gameOver == true)
                            {
                                break;
                            }
                            OpenNext(temp);
                            return temp;
                        case ConsoleKey.Spacebar:
                            if (field.cells[temp.y, temp.x].isOpen == false)
                            {
                                SetFlag(temp);
                                PrintCell(temp.y, temp.x);
                            }
                            break;
                        case ConsoleKey.Add:
                            OpenAll();
                            if (countOpenCells == 0)
                            {
                                break;
                            }
                            if (gameOver == false)
                            {
                                gameOver = true;
                                win = true;
                                Console.ForegroundColor = ConsoleColor.Magenta;
                                Console.SetCursorPosition(field.width + 1, 5);
                                Console.WriteLine("Win!!! Win!!!");
                                Console.ResetColor();
                                Console.SetCursorPosition(field.width + 1, 6);
                                Console.Write("Press any key...");
                                Console.ReadKey();
                                WriteRecords();
                                Console.Write("Press any key...");
                                Console.ReadKey();
                                break;
                            }
                            break;
                    }
                    if (gameOver == true && win == false)
                    {
                        Console.SetCursorPosition(field.width + 1, 6);
                        Console.Write("Press any key...");
                        Console.ReadKey();
                        RunGame();
                    }
                    else if (gameOver == true && win == true)
                        RunGame();
                }
            }
            void PrintResult()
            {
                if (countOpenCells > 0)
                    Console.Clear();
                Console.WriteLine();
                for (int i = 1; i < field.heigh; i++)
                {
                    for (int j = 1; j < field.width; j++)
                    {
                        if (field.cells[i, j].mine == true && field.cells[i, j].flag == false && field.cells[i, j].isOpen)
                        {
                            {
                                Console.ForegroundColor = ConsoleColor.Red;
                                Console.Write("X");
                                Console.ResetColor();
                            }
                        }
                        else if (field.cells[i, j].countMines > 0 && field.cells[i, j].flag == false && field.cells[i, j].isOpen)
                        {
                            {
                                Console.ForegroundColor = ConsoleColor.Green;
                                Console.Write($"{field.cells[i, j].countMines}");
                                if (field.cells[i, j].countMines < 1)
                                    Console.Write((char)2);
                                Console.ResetColor();
                            }
                            Console.ResetColor();
                        }
                        else if (field.cells[i, j].flag == true && field.cells[i, j].isOpen == false)
                        {
                            Console.ForegroundColor = ConsoleColor.Blue;
                            Console.Write("F");
                            Console.ResetColor();
                        }
                        else
                        {
                            if (field.cells[i, j].isOpen)
                            {
                                Console.ForegroundColor = ConsoleColor.Green;
                            }
                            Console.SetCursorPosition(j, i);
                            Console.Write((char)14);
                            Console.ResetColor();
                        }
                    }
                    Console.SetCursorPosition(field.width + 1, 2);
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.Write(countMinesInField);
                    Console.SetCursorPosition(field.width + 1, 4);
                    Console.ForegroundColor = ConsoleColor.Blue;
                    Console.Write(countFlagsInField);
                    Console.ResetColor();
                    Console.WriteLine();
                }
            }
            void PrintCell(int y, int x)
            {
                Console.SetCursorPosition(x, y);
                if (field.cells[y, x].mine == true && field.cells[y, x].flag == false && field.cells[y, x].isOpen)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.Write("X");
                    Console.ResetColor();
                    Console.SetCursorPosition(x, y);
                }
                else if (field.cells[y, x].countMines > 0 && field.cells[y, x].flag == false && field.cells[y, x].isOpen)
                {
                    Console.ForegroundColor = ConsoleColor.Green;
                    Console.Write($"{field.cells[y, x].countMines}");
                    if (field.cells[y, x].countMines < 1)
                    {
                        Console.Write((char)2);
                    }
                    Console.ResetColor();
                    Console.SetCursorPosition(x, y);
                }
                else if (field.cells[y, x].flag == true && field.cells[y, x].isOpen == false)
                {
                    Console.ForegroundColor = ConsoleColor.Blue;
                    Console.Write("F");
                    Console.ResetColor();
                    Console.SetCursorPosition(x, y);
                }
                else
                {
                    if (field.cells[y, x].isOpen)
                    {
                        Console.ForegroundColor = ConsoleColor.Green;
                    }
                    Console.Write((char)2);
                    Console.ResetColor();
                    Console.SetCursorPosition(x, y);
                }
            }
            void OpenAll()
            {
                if (countOpenCells == 0)
                {
                    return;
                }
                YX temp;
                for (temp.y = 1; temp.y < field.heigh; temp.y++)
                {
                    for (temp.x = 1; temp.x < field.width; temp.x++)
                    {
                        if (field.cells[temp.y, temp.x].flag == false)
                        {
                            OpenCell(temp);
                        }
                    }
                }
            }
            void Count(object obj)
            {
                if (gameOver == true && time == 0)
                {
                    return;
                }
                if (gameOver == false)
                {
                    YX temp = (YX)obj;
                    temp.x = Console.CursorLeft;
                    temp.y = Console.CursorTop;
                    Console.SetCursorPosition(field.width + 6, 0);
                    if (time == 60)
                    {
                        time = 0;
                        Console.SetCursorPosition(field.width + 12, 0);
                        Console.Write("  ");
                        minutes++;
                        if (minutes == 60)
                        {
                            minutes = 0;
                            Console.SetCursorPosition(field.width + 9, 0);
                            Console.Write("  ");
                            hours++;
                        }
                    }
                    Console.SetCursorPosition(field.width + 6, 0);
                    Console.Write(hours);
                    Console.SetCursorPosition(field.width + 8, 0);
                    Console.Write(":");
                    Console.SetCursorPosition(field.width + 9, 0);
                    Console.Write(minutes);
                    Console.SetCursorPosition(field.width + 11, 0);
                    Console.Write(":");
                    Console.SetCursorPosition(field.width + 12, 0);
                    Console.Write(time);
                    time = time + 1;
                    Console.SetCursorPosition(temp.x, temp.y);
                }
            }
            void WriteRecords()
            {
                Console.Clear();
                Console.Write("Enter your name:");
                GamerData gamerData;
                gamerData.name = Console.ReadLine();
                gamerData.totalSeconds = ((hours * 3600) + minutes * 60 + time) - 1;
                results.Add(gamerData);
                results.Sort(delegate (GamerData gd1, GamerData gd2)
                { return gd1.totalSeconds.CompareTo(gd2.totalSeconds); });
                string filePath = "records.dat";
                using (FileStream fs = new FileStream(filePath, FileMode.Create))
                {
                    using (BinaryWriter bw = new BinaryWriter(fs, Encoding.Unicode))
                    {
                        bw.Write(results.Count);
                        for (int i = 0; i < results.Count; i++)
                        {
                            bw.Write(results[i].name);
                            bw.Write(results[i].totalSeconds);
                        }
                    }
                }
            }
            void ReadRecords()
            {
                string filePath = "records.dat";
                using (FileStream fs = new FileStream(filePath, FileMode.OpenOrCreate))
                {
                    if (fs.Length == 0)
                    {
                        return;
                    }
                    using (BinaryReader br = new BinaryReader(fs, Encoding.Unicode))
                    {
                        int size = br.ReadInt32();
                        GamerData gamerData;
                        for (int i = 0; i < size; i++)
                        {
                            gamerData.name = br.ReadString();
                            gamerData.totalSeconds = br.ReadInt32();
                            results.Add(gamerData);
                        }
                    }
                }
            }
            int PrintStartPage()
            {
                results.Sort(delegate (GamerData gd1, GamerData gd2)
                { return gd1.totalSeconds.CompareTo(gd2.totalSeconds); });
                Console.Clear();
                Console.WriteLine("\t\tMINESWEEPER");
                Console.WriteLine("\t\t-----------");
                Console.WriteLine("\n\t    The best 10 results\n");
                int h = 0;
                int m = 0;
                int s = 0;
                for (int i = 0; i < results.Count && i < 10; i++)
                {
                    if (results[i].totalSeconds >= 3600)
                    {
                        h = results[i].totalSeconds / 3600;
                    }
                    if ((results[i].totalSeconds - (h * 3600)) > 0)
                    {
                        m = (results[i].totalSeconds - (h * 3600)) / 60;
                    }
                    s = results[i].totalSeconds - h * 3600 + m * 60;
                    Console.SetCursorPosition(0, i + 5);
                    Console.Write($"{i + 1}. {results[i].name}");
                    Console.SetCursorPosition(30, i + 5);
                    Console.WriteLine($"{ h}:{m}:{s}\n");
                }
                Console.WriteLine("\nPress any key to continue...");
                Console.WriteLine("\nPress Esc to exit...");
                switch (Console.ReadKey(intercept: true).Key)
                {
                    case ConsoleKey.Escape:
                        return 0;
                    case ConsoleKey.Enter:
                        Console.Clear();
                        return 1;
                    default:
                        Console.Clear();
                        return 1;
                }
            }
            static public void RunGame()
            {
                Game game = new Game();
                game.ReadRecords();
                int action = game.PrintStartPage();
                if (action == 0)
                {
                    string filePath = "records.dat";
                    using (FileStream fs = new FileStream(filePath, FileMode.Create))
                    {
                        using (BinaryWriter bw = new BinaryWriter(fs, Encoding.Unicode))
                        {
                            bw.Write(game.results.Count);
                            for (int i = 0; i < game.results.Count; i++)
                            {
                                bw.Write(game.results[i].name);
                                bw.Write(game.results[i].totalSeconds);
                            }
                        }
                    }
                    System.Diagnostics.Process.GetCurrentProcess().Kill();
                }
                YX temp;
                temp.x = 1;
                temp.y = 1;
                game.field.printField();
                game.PrintResult();

                while (true)
                {
                    temp = game.MoveCursor(temp.y, temp.x);
                    if (temp.x == -1 || temp.y == -1)
                    {
                        return;
                    }
                    //return;
                }
            }
        }
        static void Main(string[] args)
        {
            Game.RunGame();
        }
    }
}
