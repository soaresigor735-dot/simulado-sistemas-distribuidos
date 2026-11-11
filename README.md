# simulado-sistemas-distribuidos

import threading
import time

# Central de pedidos: dois servidores simulados.
principal = {"online": True, "estoque": 10}
replica = {"online": True, "estoque": 10}

MAXIMO_SIMULTANEO = 2  # CORRECAO A: No máximo 2 trabalhadores simultâneos
semaforo = threading.Semaphore(MAXIMO_SIMULTANEO)
trava = threading.Lock()
pedidos_confirmados = []

def processar_pedido(numero):
    with semaforo:
        time.sleep(0.1)
        with trava:
            if principal["estoque"] > 0:
                principal["estoque"] -= 1
                pedidos_confirmados.append(numero)

def sincronizar():
    replica["estoque"] = principal["estoque"]  # CORRECAO B: Copia o estoque atualizado do principal

def consultar_estoque():
    if principal["online"]:
        return "PRINCIPAL", principal["estoque"]
    if replica["online"]:  # CORRECAO C: Corrigido para verificar a réplica
        return "REPLICA", replica["estoque"]
    return "INDISPONIVEL", None

if __name__ == "__main__":
    print("=== CENTRAL DE PEDIDOS ===")
    print("Limite simultaneo:", MAXIMO_SIMULTANEO)
    print("Estoque inicial:", principal["estoque"])
    
    trabalhadores = [
        threading.Thread(target=processar_pedido, args=(n,))
        for n in [101, 102, 103]
    ]
    
    for t in trabalhadores:
        t.start()
    for t in trabalhadores:
        t.join()
        
    print("Pedidos:", sorted(pedidos_confirmados))
    print("Estoque principal:", principal["estoque"])
    print("Replica antes:", replica["estoque"])
    
    sincronizar()
    print("Replica depois:", replica["estoque"])
    
    principal["online"] = False  # Simula a falha.
    nome, estoque = consultar_estoque()
    print("Consulta apos falha:", nome, estoque)
