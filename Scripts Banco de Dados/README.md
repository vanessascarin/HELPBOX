# Banco de Dados: Scripts 
## Usuários 

	 CREATE TABLE Usuario (
	    id_User INT IDENTITY(1,1) PRIMARY KEY,
	    email_User VARCHAR(255) NOT NULL UNIQUE,
	    senha_User VARCHAR(255) NOT NULL,
	    cargo_User VARCHAR(255) NOT NULL,
	    departamento_User VARCHAR(255) NOT NULL,
	    nivelAcesso_User INT NOT NULL, -- 1 Cliente | 2 Tecnico | 3 Administrador 
	    nome_User VARCHAR(255) NOT NULL,
	    sobrenome_User VARCHAR(255) NOT NULL
	);

## CRIAÇÃO DAS TABELAS DE USUARIOS

	CREATE TABLE Cliente ( -- 1
	    id_User INT PRIMARY KEY,
	    FOREIGN KEY (id_User) REFERENCES Usuario(id_User) ON DELETE CASCADE
	);
	
	CREATE TABLE Tecnico ( -- 2
	    id_User INT PRIMARY KEY,
	    FOREIGN KEY (id_User) REFERENCES Usuario(id_User) ON DELETE CASCADE
	);
	
	CREATE TABLE Administrador ( -- 3
	    id_User INT PRIMARY KEY,
	    FOREIGN KEY (id_User) REFERENCES Usuario(id_User) ON DELETE CASCADE
	);

## TRIGGER PARA INSERIR OS USUARIOS AUTOMATICAMENTE EM SUAS TABELAS RESPECTIVAS
	CREATE TRIGGER trg_InserirUsuario 
	ON Usuario 
	AFTER INSERT
	AS 
	BEGIN
		inserir na tabela cliente
		INSERT INTO Cliente (id_User)
		SELECT id_User FROM inserted WHERE nivelAcesso_User = 1;
	
	Inserir na tabela Tecnico
	    INSERT INTO Tecnico (id_User)
	    SELECT id_User FROM inserted WHERE nivelAcesso_User = 2;
	
	Inserir na tabela Administrador
	    INSERT INTO Administrador (id_User)
	    SELECT id_User FROM inserted WHERE nivelAcesso_User = 3;
	
	END;


## Trigger para padrozinar o CASE do Cargo de acesso

	CREATE TRIGGER trg_Usuario_PadronizarCargo
	ON Usuario
	INSTEAD OF INSERT, UPDATE
	AS
	BEGIN
	    SET NOCOUNT ON;
	
	INSERT INTO Usuario (
	        email_User,
	        senha_User,
	        cargo_User,
	        departamento_User,
	        nivelAcesso_User,
	        nome_User,
	        sobrenome_User
	    )
	    SELECT 
	        email_User,
	        senha_User,
	        -- Padroniza o cargo com base no nível de acesso
	        CASE nivelAcesso_User
	            WHEN 1 THEN 'Cliente'
	            WHEN 2 THEN 'Tecnico'
	            WHEN 3 THEN 'Administrador'
	            ELSE 'Desconhecido'
	        END AS cargo_User,
	        departamento_User,
	        nivelAcesso_User,
	        nome_User,
	        sobrenome_User
	    FROM inserted;
	END;
