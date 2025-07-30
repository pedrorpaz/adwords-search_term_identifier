/**
 * @name Tooling - Anomaly Detector
 * @description Valida a estrutura das campanhas ativas em contas do MCC, identificando anomalias como:
 *              - Campanhas sem AdGroups
 *              - Campanhas com número incorreto de AdGroups (menos de 3 ou mais de 4)
 *              - (opcional) AdGroups sem anúncios móveis ou ALL
 *              Resultado exportado para aba "anomaly-1" de planilha configurada.
 *
 * @author pedrorpaz
 * @team SEM / TOOLING
 * @date 2025-07-30
 *
 * @dependencies Google Ads Scripts API, Google Sheets API
 * @spreadsheet https://docs.google.com/spreadsheets/d/1TZFSx6gi8PWdWa2tbUmAevyH3VRvTprMFbHhPbA3Eek/
 */

//------------------------------------------ Configurações ------------------------------------------
var MCCID = "ManagerCustomerId IN ARRAY_IDS";
var SPREADSHEET_URL = url;
var spreadsheet = SpreadsheetApp.openByUrl(SPREADSHEET_URL);

//------------------------------------------ Variáveis Globais ------------------------------------------
var anomalies_Log_1 = [];
var anomalies_Log_1_c = 0;

//------------------------------------------ Execução Principal ------------------------------------------
function anomalies_All() {
  anomalies_1();
  exportResults();
}

//------------------------------------------ Regra 1: Validação de AdGroups por campanha ------------------------------------------
function anomalies_1() {
  var accname = AdWordsApp.currentAccount().getName();
  var campaigns = AdWordsApp.campaigns().withCondition("Status = ENABLED").forDateRange("TODAY").get();

  while (campaigns.hasNext()) {
    var campaign = campaigns.next();
    var adGroups = campaign.adGroups().withCondition("Status = ENABLED").get();
    var count = adGroups.totalNumEntities();

    if (count === 0) {
      anomalies_Log_1.push([accname, campaign.getName(), 'No AdGroups in Campaign', ""]);
      anomalies_Log_1_c++;
    } else if (count < 3 || count > 4) {
      anomalies_Log_1.push([accname, campaign.getName(), 'Wrong Number of AdGroups', count]);
      anomalies_Log_1_c++;
    }
  }
}

//------------------------------------------ Exporta os resultados para planilha ------------------------------------------
function exportResults() {
  var sheet = spreadsheet.getSheetByName("anomaly-1");
  var lastLine = sheet.getLastRow();

  if (anomalies_Log_1.length > 0) {
    sheet.getRange("A" + (lastLine + 1) + ":D" + (anomalies_Log_1.length + lastLine)).setValues(anomalies_Log_1);
    Logger.log(anomalies_Log_1.join('\n').replace(/,/g, '\t'));
  } else {
    Logger.log("Nenhuma anomalia encontrada.");
  }
}
