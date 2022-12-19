# Spring-Security

<br>

- Spring Security 동작 원리
<br>

![image](https://user-images.githubusercontent.com/87969121/208435330-9c3f85bf-2110-414e-ab1d-50303b4edd40.png)

<br>
<br>

**[1]** 사용자가 Http Body에 username과 password를 담아 로그인 요청을 보내면 AuthenticationFilter가 제일 먼저 동작해 이 요청을 가로챈다.

**[2]** 이 필터가 위에서 사용자로부터 받은 데이터로 UsernamePaswordAuthenticationToken을 만들어준다.

**[3]** 이 토큰 객체를 AuthenticationManager에게 넘긴다. (실질적으로는 AuthenticationManager 인터페이스를 구현한 ProviderManager이다.)

**[4]** ProviderManager는 여러 AutenticationProvider들을 순회하면서 토큰 객체를 처리해줄 AutenticationProvider를 찾아 해당 토큰을 전달한다.

**[5]** 이제 UserDetailService에서 입력받은 username로 loadUserByUsername 메소드가 수행되면서 DB에 사용자 정보가 있다면 UserDetails 형으로 가져온다. 만약, 존재하지 않으면 예외를 던진다.
- UserDetailsService는 DB에서 사용자 정보를 가져오는 역할을 하고, UserDeatails는 사용자의 정보를 담는 역할을 한다.

**[6]** 사용자가 존재한다면 AuthenticationManager가 입력받은 password를 Bcrypt로 인코딩해 사용자의 password와 비교한다.
(정리하면, username만 UserDetailService에게 보내 사용자를 찾게하고 password는 AuthenticationManager가 비교한다.)

**[7]** 사용자 인증이 완료되면 Authenticaiton을 SecurityContextHolder 객체 안의 SecurityContext에 저장한다.

<br>
<br>
<br>
<br>
<br>


- 시큐리티 환경설정
> 시큐리티 설정을 할 때 WebSecurityConfigurerAdapter를 extends해서 configure 메서드를 오버라이딩해서 구현했었다. 하지만, 이제는 WebSecurityConfigurerAdapter가 Deprecated가 되어 사용할 수 없게 되었다.

그래서 이제부터는 SecurityFilterChain를 Bean으로 등록해서 사용해야 한다.
내가 작성한 코드는 다음과 같다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder encoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.authorizeRequests()
                .antMatchers("/user/**").authenticated()
                .antMatchers("/manager/**").access("hasAnyRole('ROLE_MANAGER', 'ROLE_ADMIN')")
                .antMatchers("/admin/**").access("hasRole('ROLE_ADMIN')")
                .anyRequest().permitAll()
                .and()
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/login")
                .defaultSuccessUrl("/");

        return http.build();
    }
}
```
