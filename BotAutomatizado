import requests
import csv
import time
from multiprocessing import Pool
from logging import basicConfig, warning
from datetime import date, datetime
from bs4 import BeautifulSoup
from lxml import html

h1 = {
    'Host': 'br.investing.com',
    'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0',
    'Accept': 'text/html, */*; q=0.01',
    'Accept-Language': 'pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate, br',
    'X-Requested-With': 'XMLHttpRequest',
    'DNT': '1',
    'Connection': 'keep-alive',
    'Referer': 'https://br.investing.com/currencies/single-currency-crosses',
    'Cookie': 'PHPSESSID=nviokgf6u1mtsjmpod3q6mn5gu; geoC=BR; prebid_page=0; prebid_session=0; adBlockerNewUserDomains=1584112319; nyxDorf=ODwyZmM8M3EzbDowYSxlZjdjP2EzKjY9Zmc%3D; StickySession=id.12354917567.655.br.investing.com; adbBLk=30; G_ENABLED_IDPS=google; gtmFired=OK; r_p_s_n=1'
}

links = requests.get("https://br.investing.com/currencies/single-currency-crosses", headers=h1)
bsoup = BeautifulSoup(links.content, 'html.parser')
link = bsoup.find_all('td', {'class':'bold left noWrap elp plusIconTd'})

data = str(date.today())
newData = data[0] + data[1] + data[2] + data[3]

def minerar(index):
    basicConfig(format='[%(asctime)s] - %(message)s', datefmt='%d/%b/%y %H:%M:%S')

    # String Formatter
    name = link[index].text
    newName = str()
    for w in range(len(name)):
        if (name[w] == "/"):
            newName = newName + "-"
        else:
            newName = newName + name[w]

    warning(f"Mineração da moeda {newName} iniciada")

    referer = link[index].a["href"] + "-historical-data"    

    h2 = {
        'Host': 'br.investing.com',
        'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0',
        'Accept': 'text/plain, */*; q=0.01',
        'Accept-Language': 'pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3',
        'Accept-Encoding': 'gzip, deflate, br',
        'Content-Type': 'application/x-www-form-urlencoded',
        'X-Requested-With': 'XMLHttpRequest',
        'Content-Length': '178',
        'Origin': 'https://br.investing.com',
        'DNT': '1',
        'Connection': 'keep-alive',
        'Referer': str(referer)
    }
    
    id =  1 + index
    smlID = 106682 + index
    st_date = '1979'
    end_date = '1989'
    info = []

    while True:
        params = {
            "curr_id":str(id),
            "smlID":str(smlID),
            "st_date":'01/01/'+str(st_date),
            "end_date":'31/12/'+str(end_date),
            "interval_sec":"Daily",
            "sort_col":"date",
            "sort_ord":"DESC",
            "action":"historical_data"
        }

        page = requests.post("https://br.investing.com/instruments/HistoricalDataAjax", headers=h2, data=params)
        tree = html.fromstring(page.content)

        linha = 0
        contador = 1

        while True:
            try:
                data = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[1]/text()' % contador))[0]
                ultimo = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[2]/text()' % contador))[0]
                abertura = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[3]/text()' % contador))[0]
                maxima = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[4]/text()' % contador))[0]
                minima = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[5]/text()' % contador))[0]
                var = list(tree.xpath('//*[@id="results_box"]/table/tbody/tr[%d]/td[6]/text()' % contador))[0]

                # inserir = data + ' ; ' + ultimo + ' ; ' + abertura + ' ; ' + maxima + ' ; ' + minima + ' ; ' + var
                info.append({'data': data, 'ultimo': ultimo, 'abertura': abertura, 'maxima': maxima, 'minima': minima, 'var': var})
                linha = linha + 1
                contador = contador + 1
                print (data,ultimo,abertura,maxima,minima,var)

            except IndexError: # Se não houver mais dados para minerar
                break
        
        if ((int(st_date) + 10) > int(newData)):
            st_date = str(int(newData))
        else:
            st_date = str(int(st_date) + 10)

        if (int(end_date) == int(newData)):
            info.sort(key=lambda x: datetime.strptime(x['data'], '%d.%m.%Y'))

            with open('csv/'+newName+'.csv', 'w', newline='') as csvfile:
                fieldnames = ['Data', 'Último', 'Abertura', 'Máxima', 'Mínima', 'Variação']
                spamwriter = csv.DictWriter(csvfile, fieldnames=fieldnames)
                spamwriter.writeheader()

                for i in range(len(info)):
                    spamwriter.writerow({'Data': info[i]['data'], 'Último': info[i]['ultimo'], 'Abertura': info[i]['abertura'],'Máxima': info[i]['maxima'], 'Mínima': info[i]['minima'], 'Variação': info[i]['var']})
            
            warning(f"Mineração da moeda {newName} terminada")
            break
    
        elif ((int(end_date) + 10) > int(newData)):
            end_date = str(int(newData))
            
        else:
            end_date = str(int(st_date) + 10)

        print (data , ultimo)
if __name__ == "__main__":
    indexes = range(len(link))
    p = Pool()
    p.map(minerar, indexes)
    p.close()
    p.join()
