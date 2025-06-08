 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a//dev/null b/minesweeper.py
index 0000000000000000000000000000000000000000..981ad14e1a18ba6d5d191b0898ce8ad1d5267544 100644
--- a//dev/null
+++ b/minesweeper.py
@@ -0,0 +1,132 @@
+import random
+from typing import List, Tuple
+
+class Cell:
+    def __init__(self):
+        self.is_mine = False
+        self.adjacent = 0
+        self.is_revealed = False
+        self.is_flagged = False
+
+class Minesweeper:
+    def __init__(self, rows: int, cols: int, mines: int):
+        if mines >= rows * cols:
+            raise ValueError("Number of mines must be less than number of cells")
+        self.rows = rows
+        self.cols = cols
+        self.mines = mines
+        self.board: List[List[Cell]] = [[Cell() for _ in range(cols)] for _ in range(rows)]
+        self._place_mines()
+        self._calculate_adjacency()
+        self.game_over = False
+
+    def _place_mines(self):
+        positions = random.sample(range(self.rows * self.cols), self.mines)
+        for pos in positions:
+            r, c = divmod(pos, self.cols)
+            self.board[r][c].is_mine = True
+
+    def _calculate_adjacency(self):
+        for r in range(self.rows):
+            for c in range(self.cols):
+                if self.board[r][c].is_mine:
+                    continue
+                count = 0
+                for nr, nc in self._neighbors(r, c):
+                    if self.board[nr][nc].is_mine:
+                        count += 1
+                self.board[r][c].adjacent = count
+
+    def _neighbors(self, r: int, c: int) -> List[Tuple[int, int]]:
+        neighbors = []
+        for dr in (-1, 0, 1):
+            for dc in (-1, 0, 1):
+                if dr == 0 and dc == 0:
+                    continue
+                nr, nc = r + dr, c + dc
+                if 0 <= nr < self.rows and 0 <= nc < self.cols:
+                    neighbors.append((nr, nc))
+        return neighbors
+
+    def reveal(self, r: int, c: int):
+        if self.game_over:
+            return
+        cell = self.board[r][c]
+        if cell.is_revealed or cell.is_flagged:
+            return
+        cell.is_revealed = True
+        if cell.is_mine:
+            self.game_over = True
+            return
+        if cell.adjacent == 0:
+            for nr, nc in self._neighbors(r, c):
+                if not self.board[nr][nc].is_revealed:
+                    self.reveal(nr, nc)
+
+    def toggle_flag(self, r: int, c: int):
+        if self.game_over:
+            return
+        cell = self.board[r][c]
+        if cell.is_revealed:
+            return
+        cell.is_flagged = not cell.is_flagged
+
+    def all_clear(self) -> bool:
+        for r in range(self.rows):
+            for c in range(self.cols):
+                cell = self.board[r][c]
+                if not cell.is_mine and not cell.is_revealed:
+                    return False
+        return True
+
+    def display(self):
+        for r in range(self.rows):
+            row_display = []
+            for c in range(self.cols):
+                cell = self.board[r][c]
+                if cell.is_flagged:
+                    row_display.append('F')
+                elif not cell.is_revealed:
+                    row_display.append('.')
+                elif cell.is_mine:
+                    row_display.append('*')
+                elif cell.adjacent == 0:
+                    row_display.append(' ')
+                else:
+                    row_display.append(str(cell.adjacent))
+            print(' '.join(row_display))
+        print()
+
+
+def run_game(rows=9, cols=9, mines=10):
+    game = Minesweeper(rows, cols, mines)
+    while True:
+        game.display()
+        if game.game_over:
+            print("Boom! You hit a mine.")
+            break
+        if game.all_clear():
+            print("Congratulations! You cleared the board.")
+            break
+        cmd = input("Enter command (r row col to reveal, f row col to flag): ")
+        parts = cmd.strip().split()
+        if len(parts) != 3:
+            print("Invalid command")
+            continue
+        action, r_str, c_str = parts
+        if not (r_str.isdigit() and c_str.isdigit()):
+            print("Coordinates must be integers")
+            continue
+        r, c = int(r_str), int(c_str)
+        if not (0 <= r < rows and 0 <= c < cols):
+            print("Coordinates out of bounds")
+            continue
+        if action == 'r':
+            game.reveal(r, c)
+        elif action == 'f':
+            game.toggle_flag(r, c)
+        else:
+            print("Unknown action")
+
+if __name__ == '__main__':
+    run_game()

 
EOF
)
