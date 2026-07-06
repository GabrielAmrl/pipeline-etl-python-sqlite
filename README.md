"""
PROJETO: PIPELINE AUTOMATIZADO DE ETL (EXTRAÇÃO, TRANSFORMAÇÃO E CARGA)
OBJETIVO: Extrair dados brutos de faturamento, aplicar regras de negócio e persistir em banco SQL.
AUTOR: Gabriel Amaral de Morais
"""

import pandas as pd
import sqlite3
from datetime import datetime

# ==========================================

# 1. STAGE: EXTRAÇÃO (Simulando carga de API/ERP)

# ==========================================

def extrair_dados():
print("[INFO] Iniciando fase de Extração...")

    # Simulando os dados brutos que viriam de uma API de vendas/faturamento
    dados_brutos = {
        'id_transacao': [1001, 1002, 1003, 1004, 1005],
        'codigo_cliente': ['C-404', 'C-201', 'C-404', 'C-302', 'C-101'],
        'valor_bruto': [1500.00, 2300.50, 850.00, 4100.00, 0.00], # Contém erro de registro (zero)
        'desconto': [150.00, None, 50.00, 400.00, 0.00],          # Contém valores nulos (None)
        'data_venda': ['2026-07-01', '2026-07-02', '2026-07-02', '2026-07-03', '2026-07-04']
    }

    df_raw = pd.DataFrame(dados_brutos)
    print(f"[INFO] {len(df_raw)} registros extraídos com sucesso.")
    return df_raw

# ==========================================

# 2. STAGE: TRANSFORMAÇÃO (Aplicação de Regras de Negócio)

# ==========================================

def transformar_dados(df):
print("[INFO] Iniciando fase de Transformação (Data Cleaning)...")

    # Regra 1: Tratar valores nulos no campo de desconto (substituir por 0)
    df['desconto'] = df['desconto'].fillna(0.0)

    # Regra 2: Filtrar e eliminar registros corrompidos ou zerados (valor_bruto deve ser > 0)
    df = df[df['valor_bruto'] > 0].copy()

    # Regra 3: Criar métrica de valor líquido (Métrica de Negócio)
    df['valor_liquido'] = df['valor_bruto'] - df['desconto']

    # Regra 4: Padronizar os tipos de dados e criar data de processamento (Auditoria)
    df['data_venda'] = pd.to_datetime(df['data_venda'])
    df['data_processamento'] = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    print("[INFO] Transformação concluída. Dados limpos e normalizados.")
    return df

# ==========================================

# 3. STAGE: CARGA (Persistência no Banco de Dados SQL)

# ==========================================

def carregar_dados(df):
print("[INFO] Iniciando fase de Carga no banco de dados...")

    # Criando conexão com o banco SQLite local (será criado automaticamente)
    conexao = sqlite3.connect('data_warehouse_corporativo.db')
    cursor = conexao.cursor()

    # Garantindo que a tabela final siga as restrições de schema corretas
    df.to_sql(
        name='fato_faturamento_limpo',
        con=conexao,
        if_exists='replace', # Substitui a tabela garantindo idempotência do pipeline
        index=False
    )

    conexao.commit()
    conexao.close()
    print("[SUCESSO] Pipeline executado de ponta a ponta. Dados salvos na tabela 'fato_faturamento_limpo'.")

# ==========================================

# ORQUESTRADOR DO PIPELINE

# ==========================================

if **name** == "**main**":
print("--- INICIANDO PIPELINE DE ETL ---")
dados_api = extrair_dados()
dados_tratados = transformar_dados(dados_api)
carregar_dados(dados_tratados)
print("---------------------------------")
