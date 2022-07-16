import pandas as pd
import cloudscraper
import statistics as stat

def findFinalPosOfNum(text, initialPosOfSectorNum):
    for i in range(initialPosOfSectorNum, len(text)):
        if text[i] == '"' :
            return i

def extractNumberOfSector(scrap):
    sectorExpression = "resultado.php?setor="
    lengthOfExpression = len(sectorExpression)
    initialPosOfExpression = scrap.text.find(sectorExpression)
    initialPosOfSectorNum = initialPosOfExpression + lengthOfExpression
    finalPosOfSectorNum = findFinalPosOfNum(scrap.text, initialPosOfSectorNum)
    try:
        if scrap.text.find("Nenhum papel encontrado") == -1:
            return int(scrap.text[initialPosOfSectorNum:finalPosOfSectorNum])
        else:
            print("Stock not found\n")
            return -1
    except:
        print("Failed to parse the sector number...\n")
        return -1

def extractDataFrame(stockName):
    scraper = cloudscraper.create_scraper()
    urlDetails = f"https://www.fundamentus.com.br/detalhes.php?papel={stockName}"
    numberOfSector = extractNumberOfSector(scraper.get(urlDetails))

    if numberOfSector == -1:
        url = "https://www.fundamentus.com.br/resultado.php"
        print("No sector found for this stock\n")
    else:
        url = f"https://www.fundamentus.com.br/resultado.php?setor={numberOfSector}"
        print("Sector found, specific comparison starting:\n")

    r = scraper.get(url)
    if r.text.find("Nenhum papel encontrado") == -1 and numberOfSector != -1:
        columns = ["Papel", "P/L", "Div.Yield", "Mrg Ebit", "ROE"]
        df = pd.read_html(r.text, decimal=",", thousands=".")[0][columns]
        rows = df.iloc[:, 0].values.tolist()
        df.index = rows
        df.drop('Papel', inplace=True, axis=1)
        return df
    else:
        return None

def removePercentageSignAndInt(item):
    return float(item[0:-1].replace(",", "."))

def normalize(mark):
    if mark < 1:
        mark = 1
    elif mark > 5:
        mark = 5
    return mark

def analyzePL(plColumn, stockPLValue):
    if stockPLValue <= 0:
        return 1
    else:
        stdDev = stat.stdev(plColumn)
        mean = stat.mean(plColumn)
        mark = ((stockPLValue - mean)/stdDev) * (-1) + 3
        mark = normalize(mark)
        return mark

def analyzeDY(dyColumn, stockDYValue):
    dyColumnWithoutPercentage = list(map(removePercentageSignAndInt, dyColumn))
    stockDYValue = removePercentageSignAndInt(stockDYValue)
    if stockDYValue <= 0:
        return 1
    else:
        stdDev = stat.stdev(dyColumnWithoutPercentage)
        mean = stat.mean(dyColumnWithoutPercentage)
        mark = ((stockDYValue - mean) / stdDev) + 3
        mark = normalize(mark)
        return mark

def analyzeMrgEbit(mrgEbitColumn, stockMrgEbitValue):
    mrgEbitColumnWithoutPercentage = list(map(removePercentageSignAndInt, mrgEbitColumn))
    stockMrgEbitValue = removePercentageSignAndInt(stockMrgEbitValue)
    if stockMrgEbitValue <= 0:
        return 1
    else:
        stdDev = stat.stdev(mrgEbitColumnWithoutPercentage)
        mean = stat.mean(mrgEbitColumnWithoutPercentage)
        mark = ((stockMrgEbitValue - mean) / stdDev) + 3
        mark = normalize(mark)
        return mark

def analyzeROE(ROEColumn, stockROEValue):
    ROEColumnWithoutPercentage = list(map(removePercentageSignAndInt, ROEColumn))
    stockROEValue = removePercentageSignAndInt(stockROEValue)
    if stockROEValue <= 0:
        return 1
    else:
        stdDev = stat.stdev(ROEColumnWithoutPercentage)
        mean = stat.mean(ROEColumnWithoutPercentage)
        mark = ((stockROEValue - mean) / stdDev) + 3
        mark = normalize(mark)
        return mark

def main():
    stock = str(input("Which stock do you want to pick? \n")).upper()
    df = extractDataFrame(stock)
    while (df is None):
        stock = str(input("Enter a valid stock: \n")).upper()
        df = extractDataFrame(stock)
    print(df.loc[stock,:],"\n")
    print(df,"\n")
    print("Labels: 5 = Very Good ; 4 = Good ; 3 = Ok ; 2 = Bad ; 1 = Very Bad\n")
    pl = analyzePL(df.loc[:, "P/L"], df.loc[stock, "P/L"])
    mrgEbit = analyzeMrgEbit(df.loc[:, "Mrg Ebit"], df.loc[stock, "Mrg Ebit"])
    dy = analyzeDY(df.loc[:, "Div.Yield"], df.loc[stock, "Div.Yield"])
    roe = analyzeROE(df.loc[:, "ROE"], df.loc[stock, "ROE"])
    print(f"PL = {pl:.2f}\n")
    print(f"Div. Yield = {dy:.2f}\n")
    print(f"Mrg. Ebit = {mrgEbit:.2f}\n")
    print(f"ROE = {roe:.2f}\n")
    avg = (pl + mrgEbit + dy + roe) / 4
    if avg >= 3:
        print(f"So the Average is {avg:.2f}. You should invest in {stock}.")
    else:
        print(f"So the Average is {avg:.2f}. You should not invest in {stock}.")

main()
