Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.

For example, given the following matrix:

```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```

Return 6.
  
# Solution
  
### Solution one

The DP solution proceeds row by row, starting from the first row. 
Let the maximal rectangle area at row i and column j be computed by [right(i,j) - left(i,j)]*height(i,j).
All the 3 variables left, right, and height can be determined by the information from previous row, 
and also information from the current row. So it can be regarded as a DP solution. The transition equations are:

```
left(i,j) = max(left(i-1,j), cur_left), cur_left can be determined from the current row
right(i,j) = min(right(i-1,j), cur_right), cur_right can be determined from the current row
height(i,j) = height(i-1,j) + 1, if matrix[i][j]=='1';
height(i,j) = 0, if matrix[i][j]=='0'
```

```cpp
class Solution {
public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        if(matrix.empty()) return 0;
        
        const std::size_t row_size(matrix.size()), col_size(matrix[0].size());
        vector<std::size_t> left(col_size, 0), right(col_size, col_size-1), height(col_size, 0);
        
        std::size_t max_area(0);
        for (std::size_t row = 0; row < row_size; ++row) {
            // calculate height on each cell
            calculateHeight(matrix, row, height);
            
            // for each height, calculate the leftmost index including that height
            calculateLeftmost(matrix, row, left);
            
            // for each height, calculate the rightmost index including that height
            calculateRightmost(matrix, row, right);
            
            for (std::size_t col = 0; col < col_size; ++col) {
                max_area = std::max(max_area, (right[col] + 1 - left[col]) * height[col]);
            }
        }
        
        return max_area;
    }
    
private:
    void calculateHeight(const vector<vector<char>> &matrix, 
                         const std::size_t row,
                         vector<std::size_t> & height) {
        const std::size_t col_size = matrix[0].size();
        for (std::size_t col = 0; col < col_size; ++col) {
            if (matrix[row][col] == '1') {
                ++ height[col];
            } else {
                height[col] = 0;
            }
        }
    }
    
    void calculateLeftmost(const vector<vector<char>> &matrix, 
                            const std::size_t row,
                            vector<std::size_t> & left) {
        const std::size_t col_size = matrix[0].size();
        std::size_t leftmost(0);
        for (std::size_t col = 0; col < col_size; ++col) {
            if (matrix[row][col] == '1') {
                left[col] = std::max(left[col], leftmost);
            } else {
                // matrix[row][col] == '0'
                // current height will be marked as 0
                // therefore, the leftmost including current height(0) is 0
                left[col] = 0;
                leftmost = col + 1;
            }
        }    
    }
    
    void calculateRightmost(const vector<vector<char>> &matrix, 
                            const std::size_t row,
                            vector<std::size_t> & right) {
        const std::size_t col_size = matrix[0].size();
        std::size_t rightmost(col_size - 1);
        for (std::size_t col = col_size; col-- > 0; ) {
            if (matrix[row][col] == '1') {
                right[col] = std::min(right[col], rightmost);
            } else {
                // matrix[row][col] == '0'
                // current height will be marked as 0
                // therefore, the rightmost including current height(0) is col_size -1 
                right[col] = col_size - 1;
                rightmost = col > 0 ? col - 1 : 0;
            }
        }    
    }
};
```


### Solution two

```cpp
class Solution {
public:
  int maximalRectangle(vector<vector<char>>& matrix) {
    if(matrix.empty()) return 0;
    int max_rec(0);
    const int M(matrix.size()),N(matrix[0].size());
    vector<int> dp(matrix[0].size(),0);
    for(int i=0;i<M;++i){
      for(int j=0;j<N;++j){
        dp[j] = matrix[i][j]=='0'? 0 : dp[j] + 1;
      }
      max_rec = std::max(max_rec,largestRectangleArea(dp));
    }
    return max_rec;
  }
private:
  int largestRectangleArea(vector<int>& heights) {
    heights.push_back(0);
    const std::size_t heights_size(heights.size());
    int max_area(0);
    vector<std::size_t> non_descend_heights;
    for (std::size_t i=0;i<heights_size;++i) {
      while (!non_descend_heights.empty() && heights[non_descend_heights.back()]>heights[i]) {
        int height = heights[non_descend_heights.back()];
        non_descend_heights.pop_back();
        int width = non_descend_heights.empty()? i : i-non_descend_heights.back()-1;
        max_area = std::max(max_area,height*width);
      }
      non_descend_heights.push_back(i);
    }
    heights.pop_back(); // revert back
    return max_area;
  }
};
```