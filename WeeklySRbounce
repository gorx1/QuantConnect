# a template, ready for inserting any trading logic and modifications based on SR
import numpy as np
import pandas as pd

class ModulatedDynamicFlange(QCAlgorithm):

    def Initialize(self):
        self.SetStartDate(2014, 11, 1)

        self.SetCash(1000000)
        self.AddEquity('SPY')
        self.SetBenchmark('SPY')
        self.SetBrokerageModel(AlphaStreamsBrokerageModel())
        self.SetExecution(ImmediateExecutionModel())
        self.SetPortfolioConstruction(EqualWeightingPortfolioConstructionModel())

        # a special structure allowing using Short ETFs
        self.Size = 100
        self.mSize = self.Size * (-1)
        
        self.SetWarmUp(4000)
        self.tickers = ["SPY"]
        
        # makin' structure to hold data & calculaions results
        self.data = {}
        self.Levels = pd.DataFrame(index=self.tickers, columns=['LC','HC','R','PR','MeanC','MidC','MedC','Up','SL','TP'])
        self.Traded = pd.DataFrame(index=self.tickers, columns=['LC','HC','MidC'])
        self.Finished = pd.DataFrame(index=self.tickers, columns=['LC','HC','MidC'])
        self.Traded[:][:] = True
        self.Finished[:][:] = True
        self.FirstCloses = dict.fromkeys(self.tickers, 0)
        self.TradingStartTime = None
        self.TradingStopTime = None
        
        #creating rolling windows
        for ticker in self.tickers:
            symbol = self.AddEquity(ticker, Resolution.Minute).Symbol
            self.data[symbol] = RollingWindow[TradeBar](65)
            consolidator = TradeBarConsolidator(timedelta(minutes = 30))
            self.SubscriptionManager.AddConsolidator(ticker, consolidator)
            consolidator.DataConsolidated += self.OnConsolidated
        
        # scheduing data gathering & calculations
        self.Schedule.On(self.DateRules.WeekStart(), self.TimeRules.AfterMarketOpen("SPY", 1), self.MakeCalculations)
        self.Schedule.On(self.DateRules.Every(DayOfWeek.Monday), self.TimeRules.AfterMarketOpen("SPY", 31), self.GetFirstCloses)
        self.Schedule.On(self.DateRules.WeekEnd(), self.TimeRules.BeforeMarketClose("SPY", 1), self.GoFlat)
    
    # creating rolling windows        
    def OnConsolidated(self, sender, bar):
        self.data[bar.Symbol].Add(bar)
        
    # cacculations
    def MakeCalculations(self):    
        if not all([window.IsReady for window in self.data.values()]):
            return
        self.TradingStartTime = self.Time + timedelta(minutes=29)
        self.TradingStopTime = self.Time + timedelta(minutes=361) + timedelta(days=4)
        closes = []
        opens = []
        self.Traded[:][:] = False
        self.Finished[:][:] = False
        
        for symbol in self.data.keys():
            bars = self.data[symbol]
            sym = str(symbol)
            for i in range (0,65):
                closes.append(bars[i].Close)
            for i in range (0,65):
                opens.append(bars[i].Open)
            self.Levels['LC'][sym] = LC = min(closes)
            self.Levels['HC'][sym] = HC = max(closes)
            self.Levels['R'][sym] = R = HC - LC
            self.Levels['PR'][sym] =  PR = (HC - LC) / LC * 10
            self.Levels['MidC'][sym] = MidC = R/2 + LC
            self.Levels['MedC'][sym] = MedC = np.median(sorted(closes))
            self.Levels['MeanC'][sym] = MeanC = np.mean(closes)
            self.Levels['SL'][sym] = SL = R / 6
            self.Levels['TP'][sym] = TP = SL * 18
            
            
            if ******************************:
                self.Levels['Up'][sym] = Up = True
            if ******************************:
                self.Levels['Up'][sym] = Up = False
            closes[:] = []
            opens[:] = []

    # a function with the purpose of grabbing the first close of current interval. For example,
    def GetFirstCloses(self):
        for symbol in self.data.keys():
            bars = self.data[symbol]
            sym = str(symbol)
            self.FirstCloses[sym] = bars[0].Close
    
    # closes all positions
    def GoFlat(self):
        #for symbol in self.data.keys():
        #    bars = self.data[symbol]
        #    self.EmitInsights(Insight.Price(symbol, timedelta(minutes=2800), InsightDirection.Flat))
        self.Liquidate()
        self.Traded[:][:] = True
        self.Finished[:][:] = True
        
        
    
    #def Mag(self):
        #Diff = self.WeekCloseTime - self.Time
        #mDiff = int(Diff.total_seconds() / 60)
            
    # execution block
    def OnData(self,data):
        if not all([window.IsReady for window in self.data.values()]):
            return
        # limits trading only during allowed time
        if str(self.TradingStopTime) > str(self.Time) > str(self.TradingStartTime):
            # taking data from rolling windows
            for symbol in self.data.keys():
                bars = self.data[symbol]
                sym = str(symbol)
                PH = bars[0].High
                PL = bars[0].Low
                PC = bars[0].Close
                PO = bars[0].Open
                
                #Blocks with trading logic, algos per se
                if self.Levels['Up'][sym] == True:        
                    if self.Traded['HC'][sym] == False and self.Finished['HC'][sym] == False:
                        if PL <= self.Levels['HC'][sym] and PC >= self.Levels['HC'][sym]:
                            self.MarketOrder(symbol, self.Size)
                            self.Traded['HC'][sym] = True
                    if self.Traded['HC'][sym] == True and self.Finished['HC'][sym] == False:
                        if PL <= (self.Levels['HC'][sym] - self.Levels['SL'][sym]):
                            self.MarketOrder(symbol, self.mSize)
                            self.Finished['HC'][sym] = True
                        if PH >= (self.Levels['HC'][sym] + self.Levels['TP'][sym]):
                            self.MarketOrder(symbol, self.mSize)
                            self.Finished['HC'][sym] = True
                if self.Levels['Up'][sym] == False:
                    if self.Traded['LC'][sym] == False and self.Finished['LC'][sym] == False:
                        if PH >= self.Levels['LC'][sym] and PC <= self.Levels['LC'][sym]:    
                            self.MarketOrder(isym, self.Size)
                            self.Traded['LC'][sym] = True
                    if self.Traded['LC'][sym] == True and self.Finished['LC'][sym] == False:
                        if PC >= (self.Levels['LC'][sym] + self.Levels['SL'][sym]):
                            self.MarketOrder(isym, self.mSize)
                            self.Finished['LC'][sym] = True
                
                if self.Levels['Up'][sym] == True:        
                    if self.Traded['LC'][sym] == False and self.Finished['LC'][sym] == False:
                        if PL <= self.Levels['LC'][sym] and PC >= self.Levels['LC'][sym]:
                            self.MarketOrder(symbol, self.Size)
                            self.Traded['LC'][sym] = True
                    if self.Traded['LC'][sym] == True and self.Finished['LC'][sym] == False:
                        if PL <= (self.Levels['LC'][sym] - self.Levels['SL'][sym]):
                            self.MarketOrder(symbol, self.mSize)
                            self.Finished['LC'][sym] = True
                        if PH >= (self.Levels['LC'][sym] + self.Levels['TP'][sym]):
                            self.MarketOrder(symbol, self.mSize)
                            self.Finished['LC'][sym] = True
                if self.Levels['Up'][sym] == False:
                    if self.Traded['HC'][sym] == False and self.Finished['HC'][sym] == False:       
                        if PH >= self.Levels['HC'][sym] and PC <= self.Levels['HC'][sym]:
                            self.MarketOrder(isym, self.Size)
                            self.Traded['HC'][sym] = True
                    if self.Traded['HC'][sym] == True and self.Finished['HC'][sym] == False: 
                        if PC >= (self.Levels['HC'][sym] + self.Levels['SL'][sym]): 
                            self.MarketOrder(isym, self.mSize)
                            self.Finished['HC'][sym] = True      
