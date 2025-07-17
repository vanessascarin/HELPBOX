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


## Trigger para padrozinar o CASE dos níveis de acesso

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


## CRIAÇÂO DA TABELA CHAMADO
	
	CREATE TABLE Chamado (
	    id_Cham INT IDENTITY(1,1) PRIMARY KEY,
	    
	    status_Cham VARCHAR(20) NOT NULL,
	    dataAbertura_Cham DATETIME NOT NULL, 
	    dataFechamento_Cham DATETIME NULL,
	    dataProblema DATETIME NOT NULL,
	    
	    prioridade_Cham CHAR(1) NOT NULL,
	    categoria_Cham VARCHAR(50) NOT NULL,
	    
	    descricao_Cham VARCHAR(1000) NOT NULL,
	    solucaoIA_Cham VARCHAR(1000) NULL,
	    solucaoTec_Cham VARCHAR(1000) NULL,
	    solucaoFinal_Cham VARCHAR(1000) NULL,
	    
	    tecResponsavel_Cham INT NULL,
	    
	    CONSTRAINT CK_Datas_Chamado CHECK ( --- Garante que a data do problema não seja depois da abertura do chamado
	        dataProblema <= dataAbertura_Cham AND
	        (
	            dataFechamento_Cham IS NULL OR 
	            dataAbertura_Cham <= dataFechamento_Cham
	        )
	    ),
	    
	    CONSTRAINT CK_Status_Chamado CHECK (  --- Garante que o usuario vai inserir apenas as 3 opções de status
	        LOWER(status_Cham) IN ('aberto', 'em andamento', 'fechado')
	    ),
	    
	    CONSTRAINT CK_Prioridade_Chamado CHECK ( ---- Garante que o usuario vai inserir apenas as opções de prioridades existentes 
	        UPPER(prioridade_Cham) IN ('A', 'M', 'B')
	    ),
	    
	    CONSTRAINT FK_Chamado_Tecnico FOREIGN KEY (tecResponsavel_Cham) 
	        REFERENCES Tecnico(id_User)
	);


## TRIGGER PARA PADRONIZAR O STATUS DO CHAMADO

	CREATE TRIGGER trg_NormalizaStatusChamado
	ON Chamado
	INSTEAD OF INSERT, UPDATE
	AS
	BEGIN
	    SET NOCOUNT ON;
	
	    -- Para UPDATE
	    IF EXISTS (SELECT * FROM inserted i JOIN deleted d ON i.id_Cham = d.id_Cham)
	    BEGIN
	        UPDATE c
	        SET
	            status_Cham = UPPER(LEFT(i.status_Cham,1)) + LOWER(SUBSTRING(i.status_Cham,2,LEN(i.status_Cham))),
	            dataAbertura_Cham = i.dataAbertura_Cham,
	            dataFechamento_Cham = i.dataFechamento_Cham,
	            dataProblema = i.dataProblema,
	            prioridade_Cham = UPPER(i.prioridade_Cham),
	            categoria_Cham = i.categoria_Cham,
	            descricao_Cham = i.descricao_Cham,
	            solucaoIA_Cham = i.solucaoIA_Cham,
	            solucaoTec_Cham = i.solucaoTec_Cham,
	            solucaoFinal_Cham = i.solucaoFinal_Cham,
	            tecResponsavel_Cham = i.tecResponsavel_Cham
	        FROM Chamado c
	        JOIN inserted i ON c.id_Cham = i.id_Cham;
	    END
	
	    -- Para INSERT
	    IF EXISTS (SELECT * FROM inserted i WHERE NOT EXISTS (SELECT 1 FROM Chamado c WHERE c.id_Cham = i.id_Cham))
	    BEGIN
	        INSERT INTO Chamado (
	            status_Cham,
	            dataAbertura_Cham,
	            dataFechamento_Cham,
	            dataProblema,
	            prioridade_Cham,
	            categoria_Cham,
	            descricao_Cham,
	            solucaoIA_Cham,
	            solucaoTec_Cham,
	            solucaoFinal_Cham,
	            tecResponsavel_Cham
	        )
	        SELECT
	            UPPER(LEFT(status_Cham,1)) + LOWER(SUBSTRING(status_Cham,2,LEN(status_Cham))) AS status_Cham,
	            dataAbertura_Cham,
	            dataFechamento_Cham,
	            dataProblema,
	            UPPER(prioridade_Cham) AS prioridade_Cham,
	            categoria_Cham,
	            descricao_Cham,
	            solucaoIA_Cham,
	            solucaoTec_Cham,
	            solucaoFinal_Cham,
	            tecResponsavel_Cham
	        FROM inserted;
	    END
	END;
