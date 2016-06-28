import oauth2 as oauth
import time
import urllib2 
import contextlib
import xml.etree.ElementTree as ET
import pandas as pd

url = "http://chpp.hattrick.org/chppxml.ashx"
url_req_token = "https://chpp.hattrick.org/oauth/request_token.ashx"
url_aut_token = "https://chpp.hattrick.org/oauth/authorize.aspx"
url_content = "http://chpp.hattrick.org/chppxml.ashx"


signature_method = oauth.SignatureMethod_HMAC_SHA1()

f = open("/Users/tropianhs/coding/hattrick/pyhattrick/pyhattrick/credentials.txt")
mykey = f.readline().strip()[11:]
mysecret = f.readline().strip()[14:]
conkey = f.readline().strip()[14:]
consecret = f.readline().strip()[17:]

token = oauth.Token(key=mykey, secret=mysecret)
consumer = oauth.Consumer(key=conkey, secret=consecret)

def get_series_id_from_name(series_name):

    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth.generate_nonce(),
        'oauth_timestamp': str(int(time.time())),
        'file': 'search',
        'version': 1.2,
        'searchType': 3,
        'searchString': series_name
    }

    req = oauth.Request(method="GET", url=url_content, parameters=params)
    req.sign_request(signature_method, consumer,token)

    with contextlib.closing(urllib2.urlopen(req.to_url(), timeout=10)) as x:
                # send the request
                responseData = x.read()
                #print responseData

    root = ET.fromstring(responseData)

    for child in root.findall('SearchResults'):
        for res in child.findall('Result'):
            id   = res.find('ResultID').text
            name = res.find('ResultName').text
            if (name==series_name): return int(id)

def get_teams_from_series_id(league_id):

    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth.generate_nonce(),
        'oauth_timestamp': str(int(time.time())),
        'file': 'leaguedetails',
        'version': 1.4,
        'leagueLevelUnitID': league_id,
    }

    req = oauth.Request(method="GET", url=url_content, parameters=params)
    req.sign_request(signature_method, consumer,token)

    with contextlib.closing(urllib2.urlopen(req.to_url(), timeout=10)) as x:
                # send the request
                responseData = x.read()
                #return responseData

    root = ET.fromstring(responseData)
    teams = []
    #for child in root.findall('CurrentMatchRound'):
    round = root.find('CurrentMatchRound').text

    for child in root.findall('Team'):
        id   = child.find('TeamID').text
        name = child.find('TeamName').text
        position = child.find('Position').text
        points = child.find('Points').text
        goalsFor = child.find('GoalsFor').text
        goalsAga = child.find('GoalsAgainst').text
        matches = child.find('Matches').text
        won = child.find('Won').text
        draws = child.find('Draws').text
        lost = child.find('Lost').text
        teams.append({"team_name":name,"team_id":int(id), "team_points":int(points),"team_position":int(position), 
                      "team_gFor":int(goalsFor), "team_gAga":int(goalsAga), "team_matches":int(matches),
                      "team_won":int(won),"team_draws":int(draws),"team_lost":int(lost),"round":int(round)})

    return teams

def get_last_league_match_id(team_id):
    
    matches = get_matches_from_team_id(team_id)
    league_matches = []
    for match in matches:
        if match["match_type"]==1: 
        #and match["match_status"] == "FINISHED"
            league_matches.append(match)
    
    match_id = league_matches[len(league_matches)-1]["match_id"]

    return match_id

def get_team_table(league_ids):
    '''
    get a list of league_ids and writes a dataframe with teams in the leagues. 
    '''
    df_teams = pd.DataFrame(columns=('team_name', 'team_id', 'series_id', 'team_position', 'team_points', 
                                     'team_gFor', 'team_gAga', 'team_matches', 'team_won', 'team_draw', 'team_lost'))
    
    for id in league_ids:
        teams = get_teams_from_series_id(id)
        for i,t in enumerate(teams):     
            df_teams.loc[i] = [t["team_name"],t["team_id"],id,t["team_position"],
                               t["team_points"],t["team_gFor"],t["team_gAga"],
                               t["team_matches"], t["team_won"], t["team_draws"], t["team_lost"]]
            
            
    return df_teams

def get_last_midfield_table(league_ids):
    '''
    get a list of league ids and write a dataframe with the midfield rating for the last match for each team
    '''
    df_midfield = pd.DataFrame(columns=('team_id', 'midfield_rating', 'match_id'))
    
    
    for id in league_ids:
        teams = get_teams_from_series_id(id)
        for i,team in enumerate(teams):
            team_id = team["team_id"]
            #print team_id
            match_id = get_last_league_match_id(team_id)
            #print match_id
            rat = get_ratings_from_match_id(match_id)        
            
            if rat[0]['team_id_home']==team_id:     
                df_midfield.loc[i] = [rat[0]["team_id_home"],rat[0]["midfield_home"],match_id]
            else:
                df_midfield.loc[i] = [rat[1]["team_id_away"],rat[1]["midfield_away"],match_id]
            
                            
    return df_midfield

def get_last_defence_table(league_ids):

    df_defence = pd.DataFrame(columns=('team_id', 'cen_def_rating', 'left_def_rating', 'right_def_rating', 'match_id'))
    
    for id in league_ids:
        #print id
        teams = get_teams_from_series_id(id)
        for i,team in enumerate(teams):
            team_id = team["team_id"]
            #print team_id
            match_id = get_last_league_match_id(team_id)
            #print match_id
            rat = get_ratings_from_match_id(match_id)        
            #print rat[0]['team_id_home']
            #print rat[1]['team_id_away']
            
            if rat[0]['team_id_home']==team_id:     
                df_defence.loc[i] = [rat[0]["team_id_home"],rat[0]["central_defence_home"],rat[0]["left_defence_home"],rat[0]["right_defence_home"],match_id]
            else:
                df_defence.loc[i] = [rat[1]["team_id_away"],rat[1]["central_defence_away"],rat[1]["left_defence_away"],rat[1]["right_defence_away"],match_id]
                            
    return df_defence

def get_last_attack_table(league_ids):

    df_attack = pd.DataFrame(columns=('team_id', 'cen_att_rating', 'left_att_rating', 'right_att_rating', 'match_id'))
    
    for id in league_ids:
        #print id
        teams = get_teams_from_series_id(id)
        for i,team in enumerate(teams):
            team_id = team["team_id"]
            #print team_id
            match_id = get_last_league_match_id(team_id)
            #print match_id
            rat = get_ratings_from_match_id(match_id)        
            #print rat[0]['team_id_home']
            #print rat[1]['team_id_away']
            
            if rat[0]['team_id_home']==team_id:     
                df_attack.loc[i] = [rat[0]["team_id_home"],rat[0]["central_attack_home"],rat[0]["left_attack_home"],rat[0]["right_attack_home"],match_id]
            else:
                df_attack.loc[i] = [rat[1]["team_id_away"],rat[1]["central_attack_away"],rat[1]["left_attack_away"],rat[1]["right_attack_away"],match_id]
                            
    return df_attack

def get_matches_from_team_id(team_id):

    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth.generate_nonce(),
        'oauth_timestamp': str(int(time.time())),
        'file': 'matchesarchive',
        'teamID': team_id,
    }

    req = oauth.Request(method="GET", url=url_content, parameters=params)
    req.sign_request(signature_method, consumer,token)

    with contextlib.closing(urllib2.urlopen(req.to_url(), timeout=10)) as x:
                # send the request
                responseData = x.read()
                #return responseData

    
    root = ET.fromstring(responseData)
    matches = []
    for child in root.findall('Team'):
        for res in child.findall('MatchList'):
            for match in res.findall('Match'):
                id     = match.find('MatchID').text
                type   = match.find('MatchType').text
                #status = match.find('Status').text
                date   = match.find('MatchDate').text                
                matches.append({"match_id":int(id),"match_type":int(type),#"match_status":status,
                                "match_date":date})

    return matches


def get_lineups_from_match_id(match_id):

    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth.generate_nonce(),
        'oauth_timestamp': str(int(time.time())),
        'file': 'matchdetails',
        'version': 2.7,
        'matchID': match_id,
    }

    req = oauth.Request(method="GET", url=url_content, parameters=params)
    req.sign_request(signature_method, consumer,token)

    with contextlib.closing(urllib2.urlopen(req.to_url(), timeout=10)) as x:
                # send the request                                                                                                                                                                                                            
                responseData = x.read()
                #return responseData                                                                                                                                                                                                          
    root = ET.fromstring(responseData)
    lineup = []
    for child in root.findall('Match'):
        for team in child.findall('HomeTeam'):
            team_name_home = team.find('HomeTeamName').text
            team_id_home = team.find('HomeTeamID').text
            formation_home = team.find('Formation').text
            tactic_type_home = team.find('TacticType').text
            tactic_skill_home = team.find('TacticSkill').text
            lineup.append({"formation_home":formation_home,
                           "tactic_type_home":int(tactic_type_home),
                           "tactic_skill_home":int(tactic_skill_home),
                           "team_name_home":team_name_home,
                           "team_id_home":int(team_id_home),})
       
        for team in child.findall('AwayTeam'):
            team_name_away = team.find('AwayTeamName').text
            team_id_away = team.find('AwayTeamID').text
            formation_away = team.find('Formation').text
            tactic_type_away = team.find('TacticType').text
            tactic_skill_away = team.find('TacticSkill').text
            lineup.append({"formation_home":formation_away,
                           "tactic_type_home":int(tactic_type_away),
                           "tactic_skill_home":int(tactic_skill_away),
                           "team_name_away":team_name_away,
                           "team_id_away":int(team_id_away)})     

    return lineup

def get_ratings_from_match_id(match_id):

    params = {
        'oauth_version': "1.0",
        'oauth_nonce': oauth.generate_nonce(),
        'oauth_timestamp': str(int(time.time())),
        'file': 'matchdetails',
        'version': 2.7,
        'matchID': match_id,
    }

    req = oauth.Request(method="GET", url=url_content, parameters=params)
    req.sign_request(signature_method, consumer,token)

    with contextlib.closing(urllib2.urlopen(req.to_url(), timeout=10)) as x:
                # send the request
                responseData = x.read()
                #return responseData

    
    root = ET.fromstring(responseData)
    ratings = []
    for child in root.findall('Match'):
        for team in child.findall('HomeTeam'):
            team_name_home = team.find('HomeTeamName').text
            team_id_home = team.find('HomeTeamID').text
            midfield_home = team.find('RatingMidfield').text
            rightdef_home = team.find('RatingRightDef').text
            middef_home = team.find('RatingMidDef').text
            leftdef_home = team.find('RatingLeftDef').text
            rightatt_home = team.find('RatingRightAtt').text
            midatt_home = team.find('RatingMidAtt').text
            leftatt_home = team.find('RatingLeftAtt').text
            defence_sp_home = team.find('RatingIndirectSetPiecesDef').text
            attack_sp_home = team.find('RatingIndirectSetPiecesAtt').text
            

            #team_name_home = team.find('HomeTeamName').text
            #team_name_home = team.find('HomeTeamName').text
            ratings.append({"team_name_home":team_name_home,
                            "team_id_home":int(team_id_home),
                            "midfield_home":int(midfield_home),
                            "central_defence_home":int(middef_home),
                            "left_defence_home":int(leftdef_home),
                            "right_defence_home":int(rightdef_home),
                            "central_attack_home":int(midatt_home),
                            "left_attack_home":int(leftatt_home),
                            "right_attack_home":int(rightatt_home),
                            "defence_sp_home":int(defence_sp_home),
                            "attack_sp_home":int(attack_sp_home)
                            })

        for team in child.findall('AwayTeam'):
            team_name_away = team.find('AwayTeamName').text
            team_id_away = team.find('AwayTeamID').text
            midfield_away  = team.find('RatingMidfield').text
            rightdef_away = team.find('RatingRightDef').text
            middef_away = team.find('RatingMidDef').text
            leftdef_away = team.find('RatingLeftDef').text
            rightatt_away = team.find('RatingRightAtt').text
            midatt_away = team.find('RatingMidAtt').text
            leftatt_away = team.find('RatingLeftAtt').text
            defence_sp_away = team.find('RatingIndirectSetPiecesDef').text
            attack_sp_away = team.find('RatingIndirectSetPiecesAtt').text
            
            ratings.append({"team_name_away":team_name_away,
                            "team_id_away":int(team_id_away),
                            "midfield_away":int(midfield_away),
                            "central_defence_away":int(middef_away),
                            "left_defence_away":int(leftdef_away),
                            "right_defence_away":int(rightdef_away),
                            "central_attack_away":int(midatt_away),
                            "left_attack_away":int(leftatt_away),
                            "right_attack_away":int(rightatt_away),
                            "defence_sp_away":int(defence_sp_away),
                            "attack_sp_away":int(attack_sp_away)
                            })
        
    return ratings
