
# 🚜 Projeto Spark Streaming - Monitoramento Agrícola

## 📖 Descrição

Projeto de engenharia de dados que simula um sistema de monitoramento em tempo real de equipamentos agrícolas utilizando **Apache Spark Streaming**. O sistema integra dados de geolocalização de máquinas agrícolas com informações meteorológicas, processando-os em streaming e armazenando os resultados em MongoDB.

## 🏗️ Arquitetura

```
┌─────────────────────┐         ┌─────────────────────┐
│  GeoLocationProducer│         │  WeatherProducer    │
│   (Faker)           │         │   (Faker)           │
└──────────┬──────────┘         └──────────┬──────────┘
           │                               │
           │ JSON Files                    │ JSON Files
           │                               │
           ▼                               ▼
    ┌──────────────────────────────────────────┐
    │     Spark Structured Streaming           │
    │  ┌────────────────────────────────────┐  │
    │  │  Stream 1: Geolocalização          │  │
    │  │  (equipment_id, x, y, region, ...) │  │
    │  └─────────────┬──────────────────────┘  │
    │                │                          │
    │                │  JOIN (region + time)    │
    │                │                          │
    │  ┌─────────────▼──────────────────────┐  │
    │  │  Stream 2: Meteorologia            │  │
    │  │  (region, temp, humidity, ...)     │  │
    │  └────────────────────────────────────┘  │
    │                                           │
    │  ┌────────────────────────────────────┐  │
    │  │  Transformações:                   │  │
    │  │  - Merge dos streams               │  │
    │  │  - Cálculo de risco operacional    │  │
    │  │  - Score de eficiência             │  │
    │  └────────────────────────────────────┘  │
    └──────────────────┬───────────────────────┘
                       │
                       ▼
              ┌────────────────┐
              │    MongoDB     │
              │   (NoSQL DB)   │
              └────────────────┘
```

## 🎯 Funcionalidades

### 1. Produtor de Geolocalização
- Simula 5 equipamentos agrícolas (tratores, colheitadeiras, etc.)
- Movimento em plano cartesiano 1000x1000 metros
- Dados gerados: coordenadas X/Y, velocidade, região, status, combustível, horas de motor
- Frequência: a cada 2 segundos

### 2. Produtor Meteorológico
- Gera dados climáticos para 4 regiões (NE, NO, SE, SO)
- Dados gerados: temperatura, umidade, pressão, vento, precipitação, condição, UV, visibilidade
- Frequência: a cada 3 segundos

### 3. Pipeline de Streaming
- **Ingestão**: Leitura de streams JSON em tempo real
- **Merge**: Join dos dados por região e janela temporal (±5 segundos)
- **Transformação**: Cálculo de métricas derivadas
- **Persistência**: Salvamento no MongoDB via foreachBatch

### 4. Métricas Calculadas
- **Risco Operacional**: ALTO/MÉDIO/BAIXO baseado em condições climáticas
- **Score de Eficiência**: Percentual baseado na velocidade do equipamento
- **Timestamp de Processamento**: Marcação temporal do processamento

## 🛠️ Tecnologias

- **PySpark 3.5.0**: Framework de processamento distribuído
- **Spark Structured Streaming**: Processamento de dados em tempo real
- **Faker**: Biblioteca para geração de dados sintéticos
- **MongoDB**: Banco de dados NoSQL para persistência
- **PyMongo**: Driver Python para MongoDB
- **Pandas**: Manipulação e análise de dados
- **Matplotlib**: Visualização de dados

## 📋 Pré-requisitos

### Opção 1: Google Colab (Recomendado para testes)
- Conta Google
- Nenhuma instalação local necessária

### Opção 2: Ambiente Local
```bash
# Python 3.8+
python --version

# Java 8 ou 11 (necessário para Spark)
java -version

# MongoDB (opcional - pode usar MongoDB Atlas)
mongod --version
```

## 🚀 Como Executar

### No Google Colab

1. **Faça upload do notebook**:
   - Acesse [Google Colab](https://colab.research.google.com/)
   - Faça upload do arquivo `agricultural_streaming_project.ipynb`

2. **Configure o MongoDB**:
   
   **Opção A: MongoDB Local (no Colab)**
   ```python
   # Já está configurado no notebook
   # Executa automaticamente na célula 2
   ```
   
   **Opção B: MongoDB Atlas (Recomendado)**
   ```python
   # 1. Crie conta gratuita em https://www.mongodb.com/cloud/atlas
   # 2. Crie um cluster gratuito
   # 3. Configure acesso à rede (0.0.0.0/0 para testes)
   # 4. Obtenha string de conexão
   # 5. Substitua na célula 8:
   MONGO_URI = "mongodb+srv://username:password@cluster.mongodb.net/"
   ```

3. **Execute as células sequencialmente**:
   - Execute célula por célula (Shift + Enter)
   - Ou execute todas: Runtime → Run all

4. **Monitore a execução**:
   - Acompanhe os logs dos produtores
   - Observe o processamento do Spark
   - Visualize inserções no MongoDB

### Localmente

1. **Clone o repositório**:
```bash
git clone <seu-repositorio>
cd agricultural-streaming
```

2. **Instale dependências**:
```bash
pip install pyspark==3.5.0 faker pymongo pandas matplotlib
```

3. **Inicie o MongoDB**:
```bash
# Se instalado localmente
mongod --dbpath /data/db

# Ou use MongoDB Atlas (veja configuração acima)
```

4. **Execute o notebook**:
```bash
jupyter notebook agricultural_streaming_project.ipynb
```

## 📊 Estrutura dos Dados

### Dados de Geolocalização
```json
{
  "equipment_id": "EQ001",
  "equipment_type": "Trator",
  "timestamp": "2024-12-13T10:30:00",
  "x_coordinate": 450.23,
  "y_coordinate": 678.91,
  "velocity": 5.2,
  "region": "SUDESTE",
  "status": "ATIVO",
  "fuel_level": 78.5,
  "engine_hours": 1234.5
}
```

### Dados Meteorológicos
```json
{
  "region": "SUDESTE",
  "timestamp": "2024-12-13T10:30:02",
  "temperature": 28.5,
  "humidity": 65.2,
  "pressure": 1013.2,
  "wind_speed": 12.3,
  "wind_direction": "NE",
  "precipitation": 0.0,
  "condition": "ENSOLARADO",
  "uv_index": 8,
  "visibility": 12.5
}
```

### Dados Enriquecidos (MongoDB)
```json
{
  "equipment_id": "EQ001",
  "equipment_type": "Trator",
  "timestamp": "2024-12-13T10:30:00",
  "x_coordinate": 450.23,
  "y_coordinate": 678.91,
  "velocity": 5.2,
  "region": "SUDESTE",
  "status": "ATIVO",
  "fuel_level": 78.5,
  "engine_hours": 1234.5,
  "temperature": 28.5,
  "humidity": 65.2,
  "pressure": 1013.2,
  "wind_speed": 12.3,
  "wind_direction": "NE",
  "precipitation": 0.0,
  "weather_condition": "ENSOLARADO",
  "uv_index": 8,
  "visibility": 12.5,
  "processing_time": "2024-12-13T10:30:05",
  "operational_risk": "BAIXO",
  "efficiency_score": 65.0
}
```

## 📈 Consultas e Análises

### Consultas MongoDB

```javascript
// Contar total de documentos
db.equipment_weather_data.countDocuments({})

// Buscar por equipamento específico
db.equipment_weather_data.find({"equipment_id": "EQ001"})

// Média de temperatura por região
db.equipment_weather_data.aggregate([
  {
    $group: {
      _id: "$region",
      avg_temp: { $avg: "$temperature" },
      count: { $sum: 1 }
    }
  }
])

// Equipamentos com alto risco operacional
db.equipment_weather_data.find({"operational_risk": "ALTO"})

// Eficiência média por tipo de equipamento
db.equipment_weather_data.aggregate([
  {
    $group: {
      _id: "$equipment_type",
      avg_efficiency: { $avg: "$efficiency_score" }
    }
  }
])
```

### Visualizações Disponíveis

O notebook inclui visualizações automáticas:
1. **Mapa de Trajetória**: Mostra o movimento dos equipamentos no plano cartesiano
2. **Temperatura por Região**: Boxplot da distribuição de temperatura
3. **Status dos Equipamentos**: Gráfico de pizza com distribuição de status
4. **Consumo de Combustível**: Gráfico de linha mostrando evolução do nível de combustível

## ⚙️ Configurações

### Parâmetros Ajustáveis

```python
# Duração da simulação (segundos)
SIMULATION_DURATION = 30

# Número de equipamentos
num_equipments = 5

# Intervalo de geração de dados
geo_interval = 2  # segundos
weather_interval = 3  # segundos

# Janela de join temporal
join_window = "5 seconds"

# Trigger do streaming
processing_trigger = "5 seconds"
```

## 🔧 Troubleshooting

### Problema: MongoDB não conecta
**Solução**:
```python
# Verifique se o MongoDB está rodando
# No Colab, execute:
!ps aux | grep mongod

# Se não estiver, inicie novamente:
!mongod --fork --logpath /var/log/mongodb.log --dbpath /data/db
```

### Problema: Spark não inicia
**Solução**:
```python
# Verifique versão do Java
!java -version

# Reinstale PySpark
!pip uninstall pyspark -y
!pip install pyspark==3.5.0
```

### Problema: Poucos dados no MongoDB
**Solução**:
- Aumente `SIMULATION_DURATION`
- Reduza intervalos dos produtores
- Verifique logs do Spark para erros

### Problema: Erro de memória
**Solução**:
```python
# Aumente memória do Spark
spark = SparkSession.builder \
    .config("spark.driver.memory", "4g") \
    .config("spark.executor.memory", "4g") \
    .getOrCreate()
```

## 📚 Conceitos de Engenharia de Dados

### Spark Streaming
- **Structured Streaming**: API de alto nível para processamento de streams
- **Micro-batching**: Processamento em pequenos lotes contínuos
- **Watermark**: Gerenciamento de dados atrasados
- **Stateful Processing**: Manutenção de estado entre batches

### Data Pipeline
- **Ingestão**: Captura de dados de múltiplas fontes
- **Transformação**: Limpeza, enriquecimento e agregação
- **Join Temporal**: Combinação de streams baseada em tempo
- **Persistência**: Armazenamento em banco de dados

### Boas Práticas
- **Idempotência**: Operações podem ser repetidas sem efeitos colaterais
- **Checkpointing**: Recuperação de falhas
- **Monitoramento**: Logs e métricas de processamento
- **Escalabilidade**: Arquitetura distribuída e particionada

## 🎓 Casos de Uso

Este projeto pode ser adaptado para:

1. **Agricultura de Precisão**:
   - Monitoramento de frota agrícola
   - Otimização de rotas de equipamentos
   - Correlação clima x operações

2. **Logística**:
   - Rastreamento de veículos
   - Análise de eficiência de combustível
   - Gestão de manutenção preventiva

3. **IoT Industrial**:
   - Monitoramento de sensores
   - Detecção de anomalias
   - Manutenção preditiva

4. **Smart Cities**:
   - Transporte público
   - Gestão de frotas municipais
   - Análise ambiental urbana

## 🔄 Próximas Melhorias

- [ ] Integração com Apache Kafka para produção de dados
- [ ] Dashboard em tempo real com Streamlit/Dash
- [ ] Modelos de Machine Learning para predição de manutenção
- [ ] API REST para consulta de dados
- [ ] Alertas em tempo real via email/SMS
- [ ] Integração com Power BI/Tableau
- [ ] Processamento com Delta Lake para Time Travel
- [ ] Testes unitários e de integração
- [ ] CI/CD com GitHub Actions
- [ ] Containerização com Docker

## 📄 Licença

Este projeto é de código aberto e está disponível sob a licença MIT.

## 👥 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para:
- Reportar bugs
- Sugerir novas funcionalidades
- Enviar pull requests
- Melhorar a documentação

## 📧 Contato

Para dúvidas ou sugestões, abra uma issue no repositório.

---
