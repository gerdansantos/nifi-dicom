# DICOM para NiFi

[`nifi-dicom`] (https://github.com/blezek/nifi-dicom) adiciona recursos do DICOM ao pacote [NiFi] (https://nifi.apache.org/) do Apache. Os novos [`Processors`] (https://nifi.apache.org/docs/nifi-docs/html/developer-guide.html) residem em um arquivo` nar` e estendem o NiFi com vários novos processadores. Os `Processadores` são auto-documentados, com alguns detalhes abaixo.

Os lançamentos do arquivo `nar` podem ser [baixados do GitHub] (https://github.com/blezek/nifi-dicom/releases).

## Construção

`` bash
# Executar testes
./gradlew test
# Construa o arquivo nar, em build / libs
./gradlew nar
`` ``

## Instalar

Para instalar o `nifi-dicom`, copie o arquivo` nar` no diretório `lib` da sua instalação do NiFi e reinicie o NiFi.

`` bash
cp build / libs / nifi-dicom * .nar $ NIFI_HOME / lib
`` ``

## Notas sobre criptografia

[Confidencialidade no nível de atributo do suplemento 55 do DICOM] (http://dicom.nema.org/Dicom/supps/sup55_03.pdf) abrange o procedimento adequado para a identificação de dados do DICOM e a potencial recuperação posterior da PHI original. O segredo é criptografar um conjunto de tags e incorporá-lo no DICOM não identificado. Mais tarde, com a chave ou senha adequada, os dados podem ser recuperados.

Um [controlador Nifi] (https://nifi.apache.org/docs/nifi-docs/html/user-guide.html#Controller_Services) fornecendo desidentificação usando o [PixelMedNet DICOM Cleaner] (http: //www.pixelmed. com / cleaner.html). Os atributos removidos ou modificados são criptografados usando o código [Bouncy Castle FIPS] (https://www.bouncycastle.org/fips_faq.html) (distribuído com o código-fonte).

Para manter o remapeamento do `UID` consistente, esse processador precisa estar associado a um` DeidentificationService`.



O `DeidentifyEncryptDICOM` possui estas propriedades relevantes:

* `Password`: senha usada para criptografar, necessária para descriptografia
* `Iterações`: número de iterações a serem usadas na criptografia, mais é melhor para segurança, mas custa ciclos de CPU

`DecryptReidentifyDICOM` descriptografa e reidentifica os dados DICOM. Deve usar a mesma `senha 'que` DeidentifyEncryptDICOM` ou os dados não serão recuperáveis. Tem a opção (`Aceitar novas séries`) para preservar o` SeriesInstanceUID` e `SOPInstanceUID` nos dados não identificados. Isso é útil principalmente para análises que criam novas séries e instâncias.

Propriedades relevantes:

* `Password`: senha para descriptografia, deve corresponder à senha` DeidentifyEncryptDICOM`
* `Accept new series`: se` true`, novas séries são permitidas, caso contrário são rejeitadas

## Processadores

### DeidentifyDICOM

Este processador implementa um desidentificador DICOM. O processador DeidentifyDICOM substitui tags DICOM por valores não identificados e armazena os valores.

#### Propriedades:

* `Controlador de desidentificação`: especificado o controlador de desidentificação para desidentificação DICOM
* `Gerar identificação`: crie identificadores gerados se o nome do paciente não corresponder ao arquivo CSV do identificador
* `Manter descritores`: mantém a descrição do texto e os atributos dos comentários
* `Manter descritores de séries`: mantenha a descrição das séries mesmo que todos os outros descritores sejam removidos
* `Keep protocol name`: mantém o nome do protocolo mesmo que todos os outros descritores sejam removidos
* `Manter características do paciente`: ​​mantenha as características do paciente (como pode ser necessário para os cálculos do PET SUV)
* `Manter identidade do dispositivo`: Mantenha a identidade do dispositivo
* `Manter identidade da instituição`: Manter identidade da instituição
* `Manter tags privadas`: mantenha todas as tags privadas. Se definido como 'false', todas as tags privadas não seguras serão removidas.
* `Adicionar sequência de equipamentos contribuintes`: adicione tags indicando o software usado para a identificação

#### Relacionamentos:

* `success`: todas as imagens DICOM desidentificadas serão roteadas como FlowFiles para esse relacionamento
* `not_matched`: os arquivos DICOM que não correspondem ao remapeamento do paciente são roteados para esse relacionamento
* `falha`: FlowFiles que não são imagens DICOM

Atributos #### FlowFile:

* ** N / A **: não define atributos

### ExtractDICOMTags

Este processador extrai tags DICOM da imagem DICOM e define os valores nos atributos do arquivo de fluxo.

#### Propriedades:

* `Extrair todas as tags DICOM`: Extraia todas as tags DICOM se true, somente as tags listadas se false
* `Construir nome do arquivo sugerido`: construa um nome de arquivo com o padrão 'PatientName / Modality_Date / SeriesNumber_SeriesDescription / SOPInstanceUID.dcm' com todos os caracteres inaceitáveis ​​mapeados para '_'

#### Relacionamentos:

* `success`: todas as imagens DICOM serão roteadas como FlowFiles para esse relacionamento
* `falha`: FlowFiles que não são imagens DICOM

Atributos #### FlowFile:

* ** N / A **: não define atributos

### ListenDICOM

Este processador implementa um receptor DICOM para ouvir imagens DICOM recebidas.

#### Propriedades:

* `Local Application Entity Title`: o ListenDICOM exige que as Entidades de Aplicação DICOM remotas usem esse Título AE ao enviar DICOM; o padrão é aceitar todos os títulos AE chamados
* `Porta de escuta`: a porta TCP à qual o processador ListenDICOM se ligará.

#### Relacionamentos:

* `success`: todas as novas imagens DICOM serão roteadas como FlowFiles para esse relacionamento

Atributos #### FlowFile:

* `dicom.callin
