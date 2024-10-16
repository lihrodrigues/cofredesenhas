import tkinter as tk
from tkinter import messagebox
import re
import random
import string
from datetime import datetime, timedelta


class PasswordVault:
    def __init__(self):
        self.password_history = {}  # Dicionário para armazenar o histórico de senhas
        self.expiration_days = 90  # Definir expiração de 90 dias
    
    def generate_password(self):
        # Gera uma senha aleatória de 12 caracteres
        while True:
            password = ''.join(random.choices(string.ascii_letters + string.digits + string.punctuation, k=12))
            if self.is_valid(password):
                return password
    
    def is_valid(self, password):
        # Verifica comprimento
        if len(password) != 12:
            return False
        
        # Verifica se contém pelo menos uma letra maiúscula, uma minúscula, um número e um caractere especial
        if not re.search(r'[A-Z]', password) or not re.search(r'[a-z]', password):
            return False
        if not re.search(r'[0-9]', password) or not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
            return False
        
        # Verifica se há sequência de caracteres (números ou letras)
        if re.search(r'(012|123|234|345|456|567|678|789|890)', password):
            return False
        if re.search(r'(abc|bcd|cde|def|efg|fgh|ghi|hij)', password.lower()):
            return False
        
        # Verifica se a senha contém palavras comuns ou nomes
        if self.contains_common_words(password):
            return False
        
        return True
    
    def contains_common_words(self, password):
        # Lista básica de palavras comuns, pode ser expandida
        common_words = ['password', 'admin', 'user', 'nome']
        for word in common_words:
            if word in password.lower():
                return True
        return False
    
    def is_password_expired(self, creation_date):
        # Verifica se a senha expirou após 90 dias
        return (datetime.now() - creation_date) > timedelta(days=self.expiration_days)
    
    def change_password(self, user, new_password):
        # Verifica se a nova senha é válida e não foi usada nas últimas 10 alterações
        if new_password in self.password_history.get(user, []):
            return False, "A senha já foi usada recentemente."
        
        if not self.is_valid(new_password):
            return False, "Senha inválida."
        
        # Adiciona a nova senha ao histórico e remove a mais antiga se necessário
        if user not in self.password_history:
            self.password_history[user] = []
        self.password_history[user].append(new_password)
        if len(self.password_history[user]) > 10:
            self.password_history[user].pop(0)
        
        return True, "Senha alterada com sucesso."

class PasswordApp:
    def __init__(self, root):
        self.vault = PasswordVault()
        self.root = root
        self.root.title("Cofre de Senhas")

        # Label de instrução
        self.label = tk.Label(root, text="Clique em 'Gerar Senha' para gerar uma nova senha.")
        self.label.pack(pady=10)

        # Campo de senha
        self.password_entry = tk.Entry(root, width=30)
        self.password_entry.pack(pady=10)

        # Botão para gerar senha
        self.generate_button = tk.Button(root, text="Gerar Senha", command=self.generate_password)
        self.generate_button.pack(pady=10)

        # Botão para validar senha
        self.validate_button = tk.Button(root, text="Validar Senha", command=self.validate_password)
        self.validate_button.pack(pady=10)

        # Botão para alterar senha
        self.change_button = tk.Button(root, text="Alterar Senha", command=self.change_password)
        self.change_button.pack(pady=10)

        # Label de status
        self.status_label = tk.Label(root, text="", fg="red")
        self.status_label.pack(pady=10)

    def generate_password(self):
        # Gerar senha
        password = self.vault.generate_password()
        self.password_entry.delete(0, tk.END)
        self.password_entry.insert(0, password)

    def validate_password(self):
        # Validar senha atual
        password = self.password_entry.get()
        if self.vault.is_valid(password):
            self.status_label.config(text="Senha válida!", fg="green")
        else:
            self.status_label.config(text="Senha inválida!", fg="red")

    def change_password(self):
        # Tentar alterar a senha
        user = "user1"  # Usuário fictício
        new_password = self.password_entry.get()
        success, message = self.vault.change_password(user, new_password)
        if success:
            messagebox.showinfo("Sucesso", message)
        else:
            messagebox.showerror("Erro", message)

# Criar a janela principal
root = tk.Tk()
app = PasswordApp(root)

# Iniciar o loop da interface
root.mainloop()
