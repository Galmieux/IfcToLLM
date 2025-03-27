# IfcToLLM
## 目的
　BIMに用いられるファイル（以下IFCファイル）をLLMに読み込ませて、BIMの内容をLLMに自然言語で質問し回答することが出来るか。

## 必要条件
- AWSで構築する
- 社内の情報を含めない
- IFCファイルを理解する

## 参考情報
https://note.com/growsic/n/ncac1472f39b4
https://www.jstage.jst.go.jp/article/aija/85/772/85_1377/_pdf

## 全体の処理フロー
上記のワークフローを実行するにあたり、技術的には下記を想定しています。

### IFCファイルの登録（前準備）

1. IFCファイルの解析  
ifcopenshellでIFCファイルを解析し、IfcBuilding, IfcStorey, IfcSpaceなどの建物要素を取得。

2. 要素を登録するCypherクエリの生成  
各要素（階層、部屋など）をNeo4jにノードとして登録するCypherクエリを生成。  
例：CREATE (s:IfcSpace {name: "Room101"})

3. 要素間の関係を登録するCypherクエリの生成  
「1階に部屋がある」といった要素間の関係性をNeo4jに登録するCypherクエリを生成。  
例：MATCH (f:IfcStorey {name: "1F"}) MERGE (f)-[:HAS_SPACE]->(s)

4. Cypherクエリの実行とデータ登録  
Pythonのneo4jドライバを使い、生成したCypherクエリを実行して建物要素と関係性をNeo4jに登録。

### ユーザーの質問に対するクエリ生成と回答構築

5. 自然言語の質問から検索クエリを生成  
LLMが「1階の部屋を教えて」という質問を該当するノードを検索するCypherクエリに変換。  
例：MATCH (s:IfcStorey)-[:HAS_SPACE]->(r:IfcSpace) WHERE s.name="1F" RETURN r

6. 検索クエリの実行とデータ取得  
生成したCypherクエリをNeo4jに実行し、該当するIfcSpaceノードを取得。

7. LLMによる回答生成  
取得したデータを元に、LLMが「1階には8部屋あります」といった自然言語の回答を構築。
