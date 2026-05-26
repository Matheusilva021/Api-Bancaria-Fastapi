# Api-Bancaria-Fastapi
Uma API bancária assíncrona permite processar várias requisições ao mesmo tempo, melhorando desempenho e escalabilidade.
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Dict

app = FastAPI()

# Simulação de banco de dados em memória
contas: Dict[int, dict] = {}

# Modelo para criação de conta
class ContaCreate(BaseModel):
    nome: str

# Modelo para operações bancárias
class Operacao(BaseModel):
    valor: float

# Modelo para transferência
class Transferencia(BaseModel):
    destino_id: int
    valor: float

# Criar conta
@app.post("/contas")
async def criar_conta(conta: ContaCreate):
    conta_id = len(contas) + 1

    contas[conta_id] = {
        "id": conta_id,
        "nome": conta.nome,
        "saldo": 0.0
    }

    return contas[conta_id]

# Consultar conta
@app.get("/contas/{conta_id}")
async def consultar_conta(conta_id: int):
    conta = contas.get(conta_id)

    if not conta:
        raise HTTPException(status_code=404, detail="Conta não encontrada")

    return conta

# Realizar depósito
@app.post("/deposito/{conta_id}")
async def deposito(conta_id: int, operacao: Operacao):
    conta = contas.get(conta_id)

    if not conta:
        raise HTTPException(status_code=404, detail="Conta não encontrada")

    conta["saldo"] += operacao.valor

    return {
        "mensagem": "Depósito realizado com sucesso",
        "saldo": conta["saldo"]
    }

# Realizar saque
@app.post("/saque/{conta_id}")
async def saque(conta_id: int, operacao: Operacao):
    conta = contas.get(conta_id)

    if not conta:
        raise HTTPException(status_code=404, detail="Conta não encontrada")

    if conta["saldo"] < operacao.valor:
        raise HTTPException(status_code=400, detail="Saldo insuficiente")

    conta["saldo"] -= operacao.valor

    return {
        "mensagem": "Saque realizado com sucesso",
        "saldo": conta["saldo"]
    }

# Transferência entre contas
@app.post("/transferencia/{origem_id}")
async def transferencia(origem_id: int, transferencia: Transferencia):
    origem = contas.get(origem_id)
    destino = contas.get(transferencia.destino_id)

    if not origem:
        raise HTTPException(status_code=404, detail="Conta de origem não encontrada")

    if not destino:
        raise HTTPException(status_code=404, detail="Conta de destino não encontrada")

    if origem["saldo"] < transferencia.valor:
        raise HTTPException(status_code=400, detail="Saldo insuficiente")

    origem["saldo"] -= transferencia.valor
    destino["saldo"] += transferencia.valor

    return {
        "mensagem": "Transferência realizada com sucesso",
        "saldo_origem": origem["saldo"],
        "saldo_destino": destino["saldo"]
    }

# Listar contas
@app.get("/contas")
async def listar_contas():
    return list(contas.values())

# Executar:
# uvicorn nome_do_arquivo:app --reload
