# -232-
作业提交
import tkinter as tk
from tkinter import messagebox
import numpy as np


class GomokuGame:
    def __init__(self, root):
        self.root = root
        self.root.title("五子棋游戏")
        self.root.geometry("680x560")
        self.root.resizable(False, False)

        # 游戏状态
        self.board_size = 15
        self.cell_size = 35
        self.board = np.zeros((self.board_size, self.board_size), dtype=int)  # 0:空, 1:黑, 2:白
        self.current_player = 1  # 1:黑棋, 2:白棋
        self.game_over = False
        self.moves = []  # 记录所有移动

        # 创建开始界面
        self.create_start_screen()

    def create_start_screen(self):
        """创建开始界面"""
        self.start_frame = tk.Frame(self.root, bg="#F5F5DC")
        self.start_frame.pack(fill="both", expand=True)

        title = tk.Label(
            self.start_frame,
            text="五子棋游戏",
            font=("楷体", 40, "bold"),
            fg="#8B4513",
            bg="#F5F5DC"
        )
        title.pack(pady=40)

        rules = tk.Label(
            self.start_frame,
            text="游戏规则：\n1. 黑方使用鼠标左键落子\n2. 白方使用鼠标右键落子\n3. 五子连珠即获胜",
            font=("宋体", 14),
            fg="#333333",
            bg="#F5F5DC",
            justify="left"
        )
        rules.pack(pady=20)

        btn_frame = tk.Frame(self.start_frame, bg="#F5F5DC")
        btn_frame.pack(pady=30)

        start_btn = tk.Button(
            btn_frame,
            text="开始游戏",
            font=("宋体", 16),
            bg="#8B4513",
            fg="white",
            width=12,
            height=1,
            command=self.start_game
        )
        start_btn.pack(side="left", padx=20)

        quit_btn = tk.Button(
            btn_frame,
            text="退出游戏",
            font=("宋体", 16),
            bg="#696969",
            fg="white",
            width=12,
            height=1,
            command=self.root.quit
        )
        quit_btn.pack(side="right", padx=20)

    def start_game(self):
        """开始游戏"""
        self.start_frame.destroy()
        self.create_game_board()

    def create_game_board(self):
        """创建游戏棋盘"""
        # 计算棋盘大小
        board_width = self.board_size * self.cell_size
        board_height = self.board_size * self.cell_size

        # 创建主框架
        self.main_frame = tk.Frame(self.root, bg="#DEB887")
        self.main_frame.pack(fill="both", expand=True)

        # 创建状态栏
        self.status_frame = tk.Frame(self.main_frame, height=40, bg="#DEB887")
        self.status_frame.pack(fill="x")

        self.status_label = tk.Label(
            self.status_frame,
            text="当前回合：黑棋",
            font=("宋体", 14),
            fg="black",
            bg="#DEB887"
        )
        self.status_label.pack(side="left", padx=20)

        # 创建重新开始按钮
        restart_btn = tk.Button(
            self.status_frame,
            text="重新开始",
            font=("宋体", 12),
            bg="#8B4513",
            fg="white",
            command=self.restart_game
        )
        restart_btn.pack(side="right", padx=20)

        # 创建画布
        canvas_width = board_width + 40
        canvas_height = board_height + 40
        self.canvas = tk.Canvas(
            self.main_frame,
            width=canvas_width,
            height=canvas_height,
            bg="#DEB887",
            highlightthickness=0
        )
        self.canvas.pack(pady=20)

        # 绘制棋盘
        self.draw_board()

        # 绑定鼠标事件
        self.canvas.bind("<Button-1>", self.left_click)  # 左键 - 黑棋
        self.canvas.bind("<Button-3>", self.right_click)  # 右键 - 白棋

    def draw_board(self):
        """绘制棋盘"""
        # 绘制棋盘背景
        self.canvas.create_rectangle(
            20, 20,
            20 + self.board_size * self.cell_size,
            20 + self.board_size * self.cell_size,
            fill="#DEB887",
            outline="#8B4513",
            width=2
        )

        # 绘制网格线
        for i in range(self.board_size):
            # 横线
            self.canvas.create_line(
                20, 20 + i * self.cell_size,
                    20 + (self.board_size - 1) * self.cell_size, 20 + i * self.cell_size,
                fill="#8B4513"
            )
            # 竖线
            self.canvas.create_line(
                20 + i * self.cell_size, 20,
                20 + i * self.cell_size, 20 + (self.board_size - 1) * self.cell_size,
                fill="#8B4513"
            )

        # 绘制天元和星位
        stars = [(3, 3), (3, 11), (7, 7), (11, 3), (11, 11)]
        for x, y in stars:
            self.canvas.create_oval(
                20 + x * self.cell_size - 4,
                20 + y * self.cell_size - 4,
                20 + x * self.cell_size + 4,
                20 + y * self.cell_size + 4,
                fill="#8B4513"
            )

        # 绘制已有棋子
        for i in range(self.board_size):
            for j in range(self.board_size):
                if self.board[i][j] == 1:  # 黑棋
                    self.draw_piece(i, j, "black")
                elif self.board[i][j] == 2:  # 白棋
                    self.draw_piece(i, j, "white")

    def draw_piece(self, row, col, color):
        """在指定位置绘制棋子"""
        x = 20 + col * self.cell_size
        y = 20 + row * self.cell_size
        radius = self.cell_size // 2 - 2

        self.canvas.create_oval(
            x - radius, y - radius,
            x + radius, y + radius,
            fill=color,
            outline="#333333" if color == "white" else "black"
        )

    def left_click(self, event):
        """左键点击事件 - 黑棋"""
        if self.game_over or self.current_player != 1:
            return
        self.place_piece(event.x, event.y, 1)

    def right_click(self, event):
        """右键点击事件 - 白棋"""
        if self.game_over or self.current_player != 2:
            return
        self.place_piece(event.x, event.y, 2)

    def place_piece(self, x, y, player):
        """在指定位置放置棋子"""
        # 计算棋盘坐标
        col = round((x - 20) / self.cell_size)
        row = round((y - 20) / self.cell_size)

        # 检查是否在有效范围内
        if row < 0 or row >= self.board_size or col < 0 or col >= self.board_size:
            return

        # 检查位置是否已有棋子
        if self.board[row][col] != 0:
            return

        # 放置棋子
        self.board[row][col] = player
        self.moves.append((row, col, player))

        # 绘制棋子
        self.draw_piece(row, col, "black" if player == 1 else "white")

        # 检查是否获胜
        if self.check_win(row, col, player):
            self.game_over = True
            winner = "黑棋" if player == 1 else "白棋"
            messagebox.showinfo("游戏结束", f"{winner}获胜！")
            self.status_label.config(text=f"{winner}获胜！")
            return

        # 切换玩家
        self.current_player = 3 - player  # 1->2, 2->1
        self.status_label.config(text=f"当前回合：{'黑棋' if self.current_player == 1 else '白棋'}")

    def check_win(self, row, col, player):
        """检查是否获胜"""
        # 检查方向: 水平、垂直、左上-右下对角线、右上-左下对角线
        directions = [
            [(0, 1), (0, -1)],  # 水平
            [(1, 0), (-1, 0)],  # 垂直
            [(1, 1), (-1, -1)],  # 左上-右下
            [(1, -1), (-1, 1)]  # 右上-左下
        ]

        for direction_pair in directions:
            count = 1  # 当前位置的棋子

            # 检查每个方向
            for dx, dy in direction_pair:
                temp_row, temp_col = row, col

                # 沿着方向检查
                for _ in range(4):  # 最多再检查4个位置
                    temp_row += dx
                    temp_col += dy

                    # 检查是否在边界内且是相同颜色的棋子
                    if (0 <= temp_row < self.board_size and
                            0 <= temp_col < self.board_size and
                            self.board[temp_row][temp_col] == player):
                        count += 1
                    else:
                        break

            # 如果连续5个相同棋子，则获胜
            if count >= 5:
                return True

        return False

    def restart_game(self):
        """重新开始游戏"""
        self.board = np.zeros((self.board_size, self.board_size), dtype=int)
        self.current_player = 1
        self.game_over = False
        self.moves = []
        self.canvas.delete("all")
        self.draw_board()
        self.status_label.config(text="当前回合：黑棋")


if __name__ == "__main__":
    root = tk.Tk()
    game = GomokuGame(root)
    root.mainloop()
