import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2Request;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.UserAuthenticationConverter;

import java.util.*;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

class FidelityAccessTokenConverterTest {

    @Test
    void testConvertAccessToken() {
        // Mock dependencies
        UserAuthenticationConverter userTokenConverter = mock(UserAuthenticationConverter.class);
        FidelityAccessTokenConverter accessTokenConverter = new FidelityAccessTokenConverter();
        accessTokenConverter.setUserTokenConverter(userTokenConverter);

        // Mock OAuth2AccessToken and OAuth2Authentication
        OAuth2AccessToken token = mock(DefaultOAuth2AccessToken.class);
        OAuth2Authentication authentication = mock(OAuth2Authentication.class);
        when(authentication.isClientOnly()).thenReturn(false);

        // Mock userTokenConverter behavior
        when(userTokenConverter.convertUserAuthentication(Mockito.any())).thenReturn(Collections.singletonMap("key", "value"));

        // Call the method
        Map<String, ?> result = accessTokenConverter.convertAccessToken(token, authentication);

        // Verify the result
        assertEquals("value", result.get("key"));
    }

    @Test
    void testExtractAccessToken() {
        // Create a sample map for testing
        Map<String, Object> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", "value2");

        // Mock OAuth2AccessToken
        FidelityAccessTokenConverter accessTokenConverter = new FidelityAccessTokenConverter();
        OAuth2AccessToken result = accessTokenConverter.extractAccessToken("tokenValue", map);

        // Verify the result
        assertEquals("tokenValue", result.getValue());
        assertEquals("value1", result.getAdditionalInformation().get("key1"));
        assertEquals("value2", result.getAdditionalInformation().get("key2"));
    }

    @Test
    void testExtractAuthentication() {
        // Create a sample map for testing
        Map<String, Object> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", "value2");

        // Mock dependencies
        UserAuthenticationConverter userTokenConverter = mock(UserAuthenticationConverter.class);
        when(userTokenConverter.extractAuthentication(map)).thenReturn(mock(Authentication.class));

        FidelityAccessTokenConverter accessTokenConverter = new FidelityAccessTokenConverter();
        accessTokenConverter.setUserTokenConverter(userTokenConverter);

        // Call the method
        OAuth2Authentication result = accessTokenConverter.extractAuthentication(map);

        // Verify the result
        assertEquals("value1", result.getOAuth2Request().getRequestParameters().get("key1"));
        assertEquals("value2", result.getOAuth2Request().getRequestParameters().get("key2"));
    }
}
