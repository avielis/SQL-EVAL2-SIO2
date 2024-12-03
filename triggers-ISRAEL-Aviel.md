1



# 1 premier Trigger


1) ##  Rendu

   tr_update_disponibilite_voiture
   AFTER INSERT sur la table Réservation
   Ce trigger met automatiquement à jour la disponibilité d'une voiture dans la base de données lorsqu'un client effectue une réservation.

 # CODE SQL DU TRIGGER

DELIMITER //

CREATE TRIGGER tr_update_disponibilite_voiture
AFTER INSERT ON Réservations
FOR EACH ROW
BEGIN

    UPDATE Voitures
    SET disponible = 0
    WHERE Voiture_id = NEW.Voiture_id;

END;


# 2 eme trigger

##  Rendu

2) tr_verif_age_client
   BEFORE INSERT sur la table  Réservations
   Ce trigger  vérifie l'âge d'un client(il faut qu'il minimim 21ans) avant qu'il ne puisse effectuer une réservation .
   
# CODE SQL DU TRIGGER

DELIMITER //

CREATE TRIGGER tr_verif_age_client
BEFORE INSERT ON Réservations
FOR EACH ROW
BEGIN
DECLARE age INT;


    SELECT TIMESTAMPDIFF(YEAR, date_naissance, CURDATE())
    INTO age
    FROM Clients
    WHERE client_id = NEW.client_id;

    
    IF age < 21 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le client doit avoir au moins 21 ans pour effectuer une réservation.';
    END IF;
END;
//


#  3 eme trigger

##  Rendu

tr_verif_validite_permis
BEFORE INSERT sur la table Clients
Ce trigger empêche l'insertion d'un client  si son numéro de permis de conduire ne respecte pas les règles. 

# CODE SQL DU TRIGGER

DELIMITER //

CREATE TRIGGER tr_verif_validite_permis
BEFORE INSERT ON Clients
FOR EACH ROW
BEGIN

    IF NEW.permis_conduire IS NULL OR NEW.permis_conduire = '' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le numéro de permis de conduire est obligatoire.';
    END IF;

    
    IF EXISTS (
        SELECT 1
        FROM Clients
        WHERE nu = NEW.permis_conduire
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit être unique.';
    END IF;

   
    IF LENGTH(NEW.permis_conduire) != 15 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit contenir exactement 15 caractères.';
    END IF;

  
    IF NEW.permis_conduire NOT REGEXP '^[A-Za-z0-9]{15}$' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit être alphanumérique.';
    END IF;
END;
//


# 4 eme trigger

##  Rendu


tr_verif_disponibilite_voiture
BEFORE INSERT sur la table  Réservations
 
Ce trigger :

Empêche l'insertion d'une réservation si :
La voiture n'existe pas dans la table Voitures.
La voiture n'est pas disponible (indisponible pour une réservation).

# CODE SQL DU TRIGGER
DELIMITER //

CREATE TRIGGER tr_verif_disponibilite_voiture
BEFORE INSERT ON Réservations
FOR EACH ROW
BEGIN
DECLARE voiture_disponible INT;


    SELECT disponible
    INTO voiture_disponible
    FROM Voitures
    WHERE voiture_id = NEW.voiture_id;

    IF voiture_disponible IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La voiture spécifiée n\'existe pas.';
    END IF;

    IF voiture_disponible = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La voiture n\'est pas disponible pour une réservation.';
    END IF;
END;
//



## 5 eme trigger

##  Rendu

tr_verif_chevauchement_reservation
BEFORE INSERT sur la table  Réservations
Ce trigger empêche les chevauchements de réservations pour une même voiture, garantissant qu'une voiture ne puisse pas être réservée simultanément par plusieurs clients. 


# CODE SQL DU TRIGGER
DELIMITER //

CREATE TRIGGER tr_verif_chevauchement_reservation
BEFORE INSERT ON Réservations
FOR EACH ROW
BEGIN
DECLARE voiture_disponible INT;


    IF EXISTS (
        SELECT 1
        FROM Réservations
        WHERE voiture_id = NEW.voiture_id
        AND (
            (NEW.date_debut BETWEEN date_debut AND date_fin)   
            OR (NEW.date_fin BETWEEN date_debut AND date_fin)   
            OR (date_debut BETWEEN NEW.date_debut AND NEW.date_fin)  
            OR (date_fin BETWEEN NEW.date_debut AND NEW.date_fin)   
        )
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La voiture est déjà réservée pour cette période.';
    END IF;
END;
//





