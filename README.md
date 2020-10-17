# ai-multiagent-algorithm

multiagent algorithm project in ai-2020

## Q1: Reflex agent
      def evaluationFunction(self, currentGameState, action)
        //Tính khoảng cách của ma gần nhất
        minGhostDistance = min([manhattanDistance(newPos, ghost.getPosition()) for ghost in newGhostStates])
        
        // tính khoảng cách của thức ăn gần nhất
        foodDistance = [manhattanDistance(newPos, food) for food in newFood.asList()]
        if len(foodDistance) == 0:
            minFoodDistance = 0
        else:
            minFoodDistance = min(foodDistance)
            
        //đánh giá:
        //  nếu khoảng cách giữa ma và pacman <=1 hoặc dừng di chuyển: 1 điểm
        //  nếu điểm tăng: 5 điểm
        //  nếu khoảng cách gần nhất đến thức ăn > ma: 4 điểm
        //  nếu khoảng cách gần nhất đến thức ăn < ma: 3 điểm
        if minGhostDistance <= 1 or action == Directions.STOP:
           	return 0
        if successorGameState.getScore() - currentGameState.getScore() > 0:
            return 5
        if minGhostDistance < minFoodDistance:
          	return 3
        return 4
        
## Q2: Minimax
      def getAction(self, gameState):
      
        //hàm lấy giá trị min (hành động của ma)
        def minValue(state, agentIndex, depth):
        
          //lấy các hành động của agent, nếu không có hành động thì trả về ngay evaluaionFunction (đã đến lá)
          legalActions = state.getLegalActions(agentIndex)
          if not legalActions:
            return self.evaluationFunction(state)
          
          // kiểm tra xem có phải ma cuối hay không:
          //  nếu đúng: trả về giá trị nhỏ nhất của các maxValue của pacman
          //  nếu sai: trả về giá trị nhỏ nhất của các minValue của những con ma còn lại
          if agentIndex == state.getNumAgents() - 1:
            return min(maxValue(state.generateSuccessor(agentIndex, action), depth) for action in legalActions)
          else:
            return min(minValue(state.generateSuccessor(agentIndex, action), agentIndex + 1, depth) for action in legalActions)
        
        //hàm lấy giá trị max (hành động của pacman)
        def maxValue(state, depth):
          
          //lấy các hành động của agent, nếu không có hành động thì trả về ngay evaluaionFunction
          legalActions = state.getLegalActions(0)
          if not legalActions or depth == self.depth:
            return self.evaluationFunction(state)
            
          //trả về giá trị lớn nhất của minValue của ma  
          return max(minValue(state.generateSuccessor(0, action), 0 + 1, depth + 1) for action in legalActions)
        
        //trả về giá trị cần tìm = giá trị lớn nhất giữa các action tiếp theo của pacman
        return max(gameState.getLegalActions(0), key=lambda action: minValue(gameState.generateSuccessor(0, action), 1, 1))

## Q3: AlphaBeta
      def getAction(self, gameState):
        
        //hàm lấy giá trị min (hành động của ma) 
        def minValue(gameState, agentIndex, depth, alpha, beta):
        
          //lấy các hành động của agent, nếu không có hành động thì trả về ngay evaluaionFunction (đã đến độ sâu nhất)
          legalActions = gameState.getLegalActions(agentIndex)
          if not legalActions:
            return self.evaluationFunction(gameState)
           
          v = float('inf')
          
          //tìm giá trị min giữa các hành động của agent
          for action in legalActions:
            newState = gameState.generateSuccessor(agentIndex, action)
            if agentIndex == newState.getNumAgents() - 1:
              newV = maxValue(newState, depth, alpha, beta)
            else:
              newV = minValue(newState, agentIndex + 1,depth, alpha, beta)
            v = min(v, newV)
            if v < alpha:
              return v
            beta = min(v, beta)
          
          // trả về giá trị min `
          return v
        
        // hàm lấy giá trị max (hành động của agent)
        def maxValue(gameState, depth, alpha, beta):
        
          //hàm lấy giá trị m (hành động của ma) 
          legalActions = gameState.getLegalActions(0)
          if not legalActions or depth == self.depth:
            return self.evaluationFunction(gameState)
          
          v = float('-inf')
          // tìm hành động tốt nhất
          if depth == 0:
            bestAction = legalActions[0]
          for action in legalActions:
            newState = gameState.generateSuccessor(0, action)
            newV = minValue(newState, 1, depth + 1, alpha, beta)
            
            // đánh giá, so sánh hành động
            if newV > v:
              v =newV
              if depth == 0:
                bestAction = action
              if v > beta:
                return v
              alpha = max(v, alpha)
          
          // nếu độ sâu = 0, cần trả về hành động tốt nhất của pacman, nếu không phải thì trả về giá trị max
          if depth == 0:
            return bestAction
          return v
        
        return maxValue(gameState, 0, float('-inf'), float('inf'))
        
## Q4: ExpectmaxAgent
      def getAction(self, gameState):
        
        // hàm đánh giá giá trị lớn nhất
        def expectimax_search(state, agentIndex, depth):
        
          // nếu đã duyệt hết agent
          if agentIndex == state.getNumAgents():
          
            // nếu đạt độ sâu nhất thì trả về evaluationFunction() (đã đạt đến độ sâu nhất)
            // không thì gọi hàm xuống độ sâu thấp hơn
            if depth == self.depth:
              return self.evaluationFunction(state)
            return expectimax_search(state, 0, depth + 1)
          else:
            //lấy các hành động của agent, nếu không có trả về evaluationFunction() (đã đạt đến độ sâu nhất)
            legalActions = state.getLegalActions(agentIndex)
            if len(legalActions) == 0:
              return self.evaluationFunction(state)
            
            // lấy các giá trị lớn nhất của các hành động
            next = (expectimax_search(state.generateSuccessor(agentIndex, action), agentIndex + 1, depth) for action in legalActions)
            
            // nếu là pacman thì trả về max
            // không thì trả về giá trị trung bình của các hành động
            if agentIndex == 0:
              return max(next)
            else:
              l = list(next)
              return sum(l) / len(l)
              
        // trả về giá trị lớn nhất giữa các hành động
        return max(gameState.getLegalActions(0), key=lambda x: expectimax_search(gameState.generateSuccessor(0, x), 1, 1))
     
