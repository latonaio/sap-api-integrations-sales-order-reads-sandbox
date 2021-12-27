# sap-api-integrations-sales-order-reads
sap-api-integrations-sales-order-reads は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で 受注データを取得するマイクロサービスです。    
sap-api-integrations-sales-order-reads には、サンプルのAPI Json フォーマットが含まれています。   
sap-api-integrations-sales-order-reads は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。   
https://api.sap.com/api/OP_API_SALES_ORDER_SRV_0001/overview

## 動作環境  
sap-api-integrations-sales-order-reads は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）　　

## クラウド環境での利用
sap-api-integrations-sales-order-reads は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。  

## 本レポジトリ が 対応する API サービス
sap-api-integrations-sales-order-reads が対応する APIサービス は、次のものです。

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_SALES_ORDER_SRV_0001/overview  
* APIサービス名(=baseURL): API_SALES_ORDER_SRV

## 本レポジトリ に 含まれる API名
sap-api-integrations-sales-order-reads には、次の API をコールするためのリソースが含まれています。  

* A_SalesOrder（受注 - ヘッダ）※受注の詳細データを取得するために、ToHeaderPartner、ToItem、ToItemPricingElement、ToItemScheduleLine、と合わせて利用されます。
* A_SalesOrderItem（受注 - 明細）※受注明細の詳細データを取得するために、ToItemPricingElement、ToItemScheduleLine、と合わせて利用されます。
* ToHeaderPartner（受注 - ヘッダ取引先）
* ToItem（受注 - 明細）
* ToItemPricingElement（受注 - 明細価格条件）
* ToItemScheduleLine（受注 - 明細納入日程行）

## API への 値入力条件 の 初期値
sap-api-integrations-sales-order-reads において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.SalesOrder.SalesOrder（受注番号）
* inoutSDC.SalesOrder.SalesOrderItem.SalesOrderItem（受注明細）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"Header", "Item" が指定されています。

```
"api_schema": "sap.s4.beh.salesorder.v1.SalesOrder.Created.v1",
"accepter": ["Header", "Item"],
"sales_order": "1",
"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
"api_schema": "sap.s4.beh.salesorder.v1.SalesOrder.Created.v1",
"accepter": ["All"],
"sales_order": "1",
"deleted": false
```

## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetSalesOrder(salesOrder, salesOrderItem string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "Header":
			func() {
				c.Header(salesOrder)
				wg.Done()
			}()
		case "Item":
			func() {
				c.Item(salesOrder, salesOrderItem)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```
## Output  
本マイクロサービスでは、[golang-logging-library](https://github.com/latonaio/golang-logging-library) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP 受注 の ヘッダデータ が取得された結果の JSON の例です。  
以下の項目のうち、"SalesOrder" ～ "ToHeaderPartner" は、/SAP_API_Output_Formatter/type.go 内 の Type Header {} による出力結果です。"cursor" ～ "time"は、golang-logging-library による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-sales-order-reads/SAP_API_Caller/caller.go#L58",
	"function": "sap-api-integrations-sales-order-reads/SAP_API_Caller.(*SAPAPICaller).Header",
	"level": "INFO",
	"message": [
		{
			"SalesOrder": "1",
			"SalesOrderType": "OR",
			"SalesOrganization": "1710",
			"DistributionChannel": "10",
			"OrganizationDivision": "00",
			"SalesGroup": "",
			"SalesOffice": "",
			"SalesDistrict": "",
			"SoldToParty": "17100001",
			"CreationDate": "/Date(1471392000000)/",
			"LastChangeDate": "/Date(1472774400000)/",
			"ExternalDocumentID": "",
			"LastChangeDateTime": "/Date(1472796947125+0000)/",
			"PurchaseOrderByCustomer": "gfh",
			"CustomerPurchaseOrderDate": "",
			"SalesOrderDate": "/Date(1471392000000)/",
			"TotalNetAmount": "52.65",
			"OverallDeliveryStatus": "C",
			"TotalBlockStatus": "",
			"OverallOrdReltdBillgStatus": "",
			"OverallSDDocReferenceStatus": "",
			"TransactionCurrency": "USD",
			"SDDocumentReason": "",
			"PricingDate": "/Date(1471392000000)/",
			"PriceDetnExchangeRate": "1.00000",
			"RequestedDeliveryDate": "/Date(1471392000000)/",
			"ShippingCondition": "01",
			"CompleteDeliveryIsDefined": false,
			"ShippingType": "",
			"HeaderBillingBlockReason": "",
			"DeliveryBlockReason": "",
			"IncotermsClassification": "EXW",
			"CustomerPriceGroup": "",
			"PriceListType": "",
			"CustomerPaymentTerms": "0004",
			"PaymentMethod": "",
			"ReferenceSDDocument": "",
			"ReferenceSDDocumentCategory": "",
			"CustomerAccountAssignmentGroup": "01",
			"AccountingExchangeRate": "0.00000",
			"CustomerGroup": "01",
			"AdditionalCustomerGroup1": "",
			"AdditionalCustomerGroup2": "",
			"AdditionalCustomerGroup3": "",
			"AdditionalCustomerGroup4": "",
			"AdditionalCustomerGroup5": "",
			"CustomerTaxClassification1": "",
			"TotalCreditCheckStatus": "",
			"BillingDocumentDate": "/Date(1471392000000)/",
			"to_Partner": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_SALES_ORDER_SRV/A_SalesOrder('1')/to_Partner",
			"to_Item": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_SALES_ORDER_SRV/A_SalesOrder('1')/to_Item"
		}
	],
	"time": "2021-12-27T15:01:24.163719+09:00"
}
```


